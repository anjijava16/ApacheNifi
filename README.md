# ApacheNifi

# https://www.youtube.com/watch?v=k-PbR0VJ6do (what is nifi)
# https://www.youtube.com/watch?v=KXKpxXIWdDE (: NiFi - Practicals )
# https://www.youtube.com/watch?v=Y5znvcJ_NWo&list=PLHre9pIBAgc4e-tiq9OIXkWJX8bVXuqlG
# https://www.youtube.com/watch?v=9OCNVt4UanA



https://blogs.apache.org/nifi/entry/stream_processing_nifi_and_spark


import org.apache.nifi.remote.client.SiteToSiteClient;
import org.apache.nifi.remote.client.SiteToSiteClientConfig;
import org.apache.nifi.spark.NiFiDataPacket;
import org.apache.nifi.spark.NiFiReceiver;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.storage.StorageLevel;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;






System.setProperty("hadoop.home.dir", "D:/hadoop");
System.out.println(System.getProperty("hadoop.home.dir"));

SiteToSiteClientConfig config = new SiteToSiteClient.Builder()
.url("http://localhost:8080/nifi")
.portName("spark")
.buildConfig();

System.out.println("%%%%%%%%%%%%%%%%%%%%%%%%"+ config.getPreferredBatchCount());

SparkConf sparkConf = new SparkConf().setAppName("NiFi-Spark Streaming example").setMaster("local[*]");
System.out.println("creating spark config"+sparkConf);
JavaStreamingContext ssc = new JavaStreamingContext(sparkConf, new Duration(1000L));

// Create a JavaReceiverInputDStream using a NiFi receiver so that we can pull data from 
// specified Port
JavaReceiverInputDStream packetStream = 
ssc.receiverStream(new NiFiReceiver(config, StorageLevel.MEMORY_ONLY()));

System.out.println(packetStream+"$$$$$$$$$$$$$$$$$$");
packetStream.count().print();

// Map the data from NiFi to text, ignoring the attributes
// JavaDStream text = packetStream.map(new Function<NiFiDataPacket,String>() {
// public String call(NiFiDataPacket dataPacket) throws Exception {
// return new String(dataPacket.getContent(), StandardCharsets.UTF_8);
// }
// });

// Extract the 'uuid' attribute
JavaDStream text = packetStream.map(new Function<NiFiDataPacket,String>() {
public String call(final NiFiDataPacket dataPacket) throws Exception {
System.out.println("data **********"+dataPacket.getContent());
System.out.println("$$$$$$$$$$$$$$$$$$$$");
return dataPacket.getAttributes().get("uuid");
}
});

//packetStream.map(dataPacket => new String(dataPacket.getContent, StandardCharsets.UTF_8));


packetStream.foreach(new Function<NiFiDataPacket, String>() {

public String call(final NiFiDataPacket arg0) throws Exception {

System.out.println(arg0.getContent().toString());
System.out.println("======+"+arg0.getAttributes());
;
return arg0.getContent().toString();
}
});
System.out.println("Done");



<dependency>
<groupId>org.apache.spark</groupId>
<artifactId>spark-sql_2.11</artifactId>
<version>1.6.0</version>
</dependency>
<dependency>
<groupId>org.apache.spark</groupId>
<artifactId>spark-hive_2.11</artifactId>
<version>1.6.0</version>
</dependency>
<dependency>
<groupId>org.apache.spark</groupId>
<artifactId>spark-mllib_2.11</artifactId>
<version>1.2.0</version>
</dependency>
<dependency>
<groupId>org.apache.spark</groupId>
<artifactId>spark-streaming_2.11</artifactId>
<version>1.6.0</version>
</dependency>
<dependency>
<groupId>org.apache.nifi</groupId>
<artifactId>nifi-spark-receiver</artifactId>
<version>0.0.2-incubating</version>
</dependency>

