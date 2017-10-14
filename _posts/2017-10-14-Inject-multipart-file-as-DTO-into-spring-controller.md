---
layout: post
title: Inject Multipart file as DTO into Spring Controller
---

Suppose you have a Sprint controller which accepts a multipart Excel file, which you need to parse and process.
One way doing this is adding a Multipart argument to your controller method, using a library like apache poi to parse the excel file and extract data from it, and finally to do something with the data. Wouldn't it be nice if the Excel data were passed into your controller method as a DTO, instead of a multipart file, this way you wouldn't have to worry about parsing the excel file yourself.

This post describes the parts you need to accomplish this.

## Custom Controller method argument resolver

The first thing you need to be able to do is inject custom arguments into controller methods.
Spring provides a way to do this. You an register additional HandlerMethodArgumentResolvers to the WebMvcConfigurerAdapter.

The following post describes a simple example:
> <https://blog.jdriven.com/2016/06/spicy-spring-inject-custom-method-argument-spring-mvc/>

## Extract multipart file from request

Next, we need to be able to access the request's multipart file in this HandlerMethodArgumentResolver.

The best documentation I found of doing this, seems to be the Spring source code. Since Spring provides a resolver which gives you easy access to the Mutlipart file inside a request, there might not be a lot of people which create their own resolver for this.

There is a MultipartResolutionDelegate, which seems to handle the extraction of the multipart object from the request.

> <https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/multipart/support/MultipartResolutionDelegate.java>

This class is used in Spring's RequestPartMethodArgumentResolver class, which is responsible for resolving the controller's method argument to a Multipart object.

> <https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/mvc/method/annotation/RequestPartMethodArgumentResolver.java>

Using these Spring classes, we should be able to construct our own HandlerMethodArgumentResolver which, given our own custom annotation on the controller method's argument, extracts the multipart file from the request.

## Convert the Excel to a DTO (or DTOs)

Next, we need to convert the Excel file we extracted from the request into a DTO or a list of DTOs.

There are github projects which resolve Excel files to DTOs.

The first such project is the excel-object-mapping project from Pramoth Suwanpech.

> <https://github.com/pramoth/excel-object-mapping>

This project provides an easy way to map an Excel file to a list of DTOs.
In order to do this, you need to annotate each field in your dto with a column name from the excel file, so that the library knows where to get it's data from.

I don't think there is much in the way of error handling either.
But it is a good staring point. The basics seem to be there.

A second project Spring's own spring-batch-extensions project.

> <https://github.com/spring-projects/spring-batch-extensions>

This project provides extensions for the Spring Batch project. The down side of this is that it uses Spring Batch specific classes like an ItemReader. The plus side is that the code seems very clean and modular, so it should be fairly easy to reuse parts of it.

The following is an experiment, demonstrating how the spring-batch-extensions can be used to parse an excel file.

```kotlin
package be.jipo.spring

import org.springframework.batch.item.ExecutionContext
import org.springframework.batch.item.excel.Sheet
import org.springframework.batch.item.excel.mapping.BeanWrapperRowMapper
import org.springframework.batch.item.excel.poi.PoiItemReader
import org.springframework.batch.item.excel.support.rowset.*
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.core.io.ClassPathResource
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController
import java.io.Closeable
import java.util.*

class BatchEnrollDto {
    lateinit var userId: String
    var learningUnitId: Double = 0.0
    lateinit var role: Date
    override fun toString() =
            "BatchEnrollD[userId=$userId, learningUnitId=$learningUnitId, role=$role]"
}

class NonEmptyRowNumberColumnNameExtractor(val headerRowNumber: Int = 0) : ColumnNameExtractor {

    override fun getColumnNames(sheet: Sheet) =
            sheet.getRow(headerRowNumber)
                    .filter { it.isNotBlank() }
                    .toTypedArray()

}

class CustomRowSetMetaData(private val sheet: Sheet) : RowSetMetaData {

    lateinit var columnNameExtractor: ColumnNameExtractor

    override fun getColumnNames() = columnNameExtractor.getColumnNames(sheet)

    override fun getColumnName(idx: Int) = columnNames[idx]

    override fun getColumnCount() = sheet.numberOfColumns

    override fun getSheetName() = sheet.name

}

class CustomRowSet(private val sheet: Sheet, private val metaData: CustomRowSetMetaData, private val ignoreEmptyRows: Boolean = false) : RowSet {

    private var currentRowIndex = -1
    private var currentRow: Array<String>? = null

    override fun next(): Boolean {
        currentRow = null
        currentRowIndex++
        if (currentRowIndex < sheet.numberOfRows) {
            currentRow = sheet.getRow(currentRowIndex)
            return if (ignoreEmptyRows && currentRow!!.all { it.isEmpty() }) next() else true
        }
        return false
    }

    override fun getMetaData() = metaData

    override fun getCurrentRowIndex() = currentRowIndex

    override fun getCurrentRow() = currentRow

    override fun getColumnValue(idx: Int) = currentRow!![idx]

    override fun getProperties(): Properties {
        val names = metaData.columnNames ?: throw IllegalStateException("Cannot create properties without meta data")

        val props = Properties()
        for (i in names.indices) {
            val value = currentRow!![i]
            if (value != null) {
                props.setProperty(names[i], value)
            }
        }
        return props
    }
}

class CustomRowSetFactory(private val columnNameExtractor: ColumnNameExtractor = RowNumberColumnNameExtractor()) : RowSetFactory {

    override fun create(sheet: Sheet): RowSet {
        val metaData = CustomRowSetMetaData(sheet)
        metaData.columnNameExtractor = columnNameExtractor
        return CustomRowSet(sheet, metaData, true)
    }

}

@Configuration
class ExcelConfig {

    @Bean
    fun excelItemReader() =
        PoiItemReader<BatchEnrollDto>().apply {
            setResource(excelFile())
            setLinesToSkip(1)
            setRowMapper(excelRowMapper())
            setRowSetFactory(rowSetFactory())
        }

    private fun excelFile() = ClassPathResource("batchEnrollTest2.xlsx")

    private fun rowSetFactory() =
            CustomRowSetFactory(NonEmptyRowNumberColumnNameExtractor())

    private fun excelRowMapper() =
        BeanWrapperRowMapper<BatchEnrollDto>().apply {
            setTargetType(BatchEnrollDto::class.java)
            setStrict(false)
        }

}

@RestController
class ExcelController(val excelItemReader: PoiItemReader<BatchEnrollDto>) {

    @GetMapping("/excel")
    fun excel(): String {
        ExcelReader(excelItemReader).use { reader ->
            reader.forEach { dto ->
                println(dto)
            }
        }

        return "finished"
    }

}

class ExcelReader<out T>(private val excelItemReader: PoiItemReader<T>): Closeable, Iterable<T> {

    init {
        excelItemReader.open(ExecutionContext())
    }

    override fun close() {
        excelItemReader.close()
    }

    override fun iterator() =
            ExcelItemReaderIterator(excelItemReader)

    inner class ExcelItemReaderIterator(excelItemReader: PoiItemReader<T>): Iterator<T> {

        private var next: T? = excelItemReader.read()

        override fun hasNext() = next != null

        override fun next(): T {
            val result = next
            next = excelItemReader.read()
            return result!!
        }

    }
}
```

This gets the job done, but the ExcelToDto HandlerMethodArgumentResolver I am invisioning differs in some ways.

The PoiItemReader uses the headers from the excel and expects the DTO to contain fields with these names. I would like to do the opposite, I would like the fields of the DTO to be used and expect the excel to contain headers with these names. Optionally, the header names could be customized using annotations on the DTO (like the excel-object-mapping project does), or using the custom annotation (e.g. @Excel) used by our HandlerMethodArgumentResolver.

The error handling of the PoiItemReader might not suffice either. Preferably, the ExcelToDto HandlerMethodArgumentResolver returns rather specific error messages, mentioning what went wrong. Some examples
* not an excel file
* missing headers (e.g. dto contains id, name, description, while excel only contains id and name)
* invalid data type (e.g. dto contains date: Date field, but excel contains 'date' column containing text which cannot be mapped to a date)

This kind of detailed error handling might be a nice to have though. The first job might be to get the thing to work in the first place.
