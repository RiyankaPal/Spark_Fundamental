## HOW TO WRITE DATAFRAME TO DISK IN SPARK:
 **General Syntax:**
 DataFrameWriter.format()\
                .option()\
                .partitionBy()\
                .bucketBy()\
                .save()

**Example:**
df.write.format("csv")\
        .option("header","true")\
        .option("mode","overwrite")\
        .option("path","....")\
        save()

**Modes in DataFrame Writer API:**
1. Append :Append mode is used to add new data to an existing data set without affecting the existing data.
2. Overwrite: Overwrite mode deletes the existing data and writes new data in its place.
3. errorIfExists : ErrorIfExists mode throws an error if the target data location already exists.
4. ignore : Ignore mode does nothing if the target data location already exists.
