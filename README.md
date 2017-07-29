DFSIO using Spark

Currently writes & reads. First write and then read the files.

compile:

	 mvn clean package

Setup:

0. Install dcos cli
1. Create a mesos orchestrated cluster using Azure container services from the portal or arm templategit init
2. Add spark service to dcos
3. Add ADLS account to the resource group
4. Grant a service principal access to the adls account. Note down the <credential> <clientid> <refreshurl> for the principal (for adls read writes)
5. Add a storage account and note the name and access key (for blob read writes)
6. Make a public folder on the blob storage and upload the jar-with-dependencies from above compile output

7. Do an ssh tunnel to Azure Container Service

   sudo ssh -i ~/.ssh/id_rsa  -fNL 80:localhost:80 -p 2200 <user>@<acsname>.centralus.cloudapp.azure.com

8. Now, access dc/os dashboard at localhost, mesos at localhost/mesos and marathon ...

9. Run the spark job using the commands below

10. Read the output using
    dcos spark log driver-<id> --file=stdout --lines_count=100



Run:

 The program writes a text file of specified size per partition to a location and takes 5 parameters plus the ones for adls or wasb
 
     1. number_of_files 
     
     2. file_size_in_bytes 
     
     3. file_write_location 
     
     4. buffer_size
     
     5. read or write or both

     for ADLS:

     6. credential
     7. clientid
     8. refreshurl

     for WASB:
     6. storage_account
     7. storage_key
     

Sample Runs on Spark on Mesos:


Write/Read 10 files in parallel of 1000000000 bytes each


dcos spark run --submit-args="--conf spark.executor.cores=1 --conf spark.executor.memory=1g --conf spark.cores.max=30 --class com.msft.dfsio.DFSIOSparkBlob https://00rmve.blob.core.windows.net/jars/spark-dfsio-1.0-SNAPSHOT-jar-with-dependencies.jar 30 1000000 wasb://dfsio@rmpub.blob.core.windows.net/dfsio 4096 write <storage_account> <storage_key>"

dcos spark run --submit-args="--conf spark.executor.cores=1 --conf spark.executor.memory=1g --conf spark.cores.max=30 --class com.msft.dfsio.DFSIOSparkBlob https://00rmvek4.blob.core.windows.net/jars/spark-dfsio-1.0-SNAPSHOT-jar-with-dependencies.jar 30 1000000 wasb://dfsio@rmpub.blob.core.windows.net/dfsio 4096 read <storage_account> <storage_key>"

dcos spark run --submit-args="--conf spark.executor.cores=1 --conf spark.cores.max=10 --class com.msft.dfsio.DFSIOSparkADLS https://00rmve.blob.core.windows.net/jars/spark-dfsio-1.0-SNAPSHOT-jar-with-dependencies.jar 10 1000000 adl://sparkdcos.azuredatalakestore.net/dfsio 4096 write <credential> <clientid> <refreshurl>"

dcos spark run --submit-args="--conf spark.executor.cores=1 --conf spark.cores.max=10 --class com.msft.dfsio.DFSIOSparkADLS https://00rmve.blob.core.windows.net/jars/spark-dfsio-1.0-SNAPSHOT-jar-with-dependencies.jar 10 1000000 adl://sparkdcos.azuredatalakestore.net/dfsio 4096 read <credential> <clientid> <refreshurl>"

Sample Output

	================================IO write benchmarks====================================
	Benchmark: Number of files : 10
	Benchmark: Size of each file : 10000000 Bytes
	Benchmark: File write location : adl://sprkdfsio.azuredatalakestore.net/dfsio
	Benchmark: Write buffer size : 4096
	Benchmark: Total write size : 100000000 Bytes
	Benchmark: Number of file write times collected : 10
	Benchmark: Fastest file write time : 0.231 s
	Benchmark: Slowest file write time : 0.316 s
	Benchmark: File write time range (Fastest - Slowest) : 85 milliseconds
	Benchmark: Total write time (Sum time across all files) : 2.627 s
	Benchmark: Average write time : 0.2627 s
	Benchmark: Median write time  : 0.255 s
	Benchmark: Variance on write time  : 911.2222222222222
	Benchmark: Standard Deviation on write time  : 0.03018645759644914 s
	Benchmark: Histogram on write time  : Map(231 -> 1, 236 -> 2, 237 -> 1, 249 -> 1, 261 -> 1, 273 -> 1, 292 -> 1, 296 -> 1, 316 -> 1)
	Benchmark: write start wall time epoch : 1500058180894
	Benchmark: write end Wall time epoch : 1500058183080
	Benchmark: Overall observed write wall time : 2.186 s
	Benchmark: Overall observed write throughput : 0.36596525 Gigabits per second
	================================IO write benchmarks====================================



