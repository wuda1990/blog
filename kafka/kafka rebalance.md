# kafka rebalance

if some consumer can't poll the data in time, the group coordinator will trigger a reblance activity, and take away the
partitions from this consumer. In such case, you would want to commit whatever you have processed before the ownership
of the partition is taken away form you.Obviously, we need do 2 things.

1. To know when a rebalance is trigger.
2. Before the rebalance, commit a particular offset.

Kafka Api allows us to specify a *ConsumerRebalnce* class, which offers two methods, *OnPartitionsRevoked*, *
onPartitionsAssigned*.
Below is the consumer code:

``` 
import java.util.*;
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.*;

public class RandomConsumer {

    public static void main(String[] args) throws Exception {

        String topicName = "RandomProducerTopic";
        KafkaConsumer<String, String> consumer = null;

        String groupName = "RG";
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092,localhost:9093");
        props.put("group.id", groupName);
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("enable.auto.commit", "false");

        consumer = new KafkaConsumer<>(props);
        RebalanceListnerrebalanceListner = new RebalanceListner(consumer);

        consumer.subscribe(Arrays.asList(topicName), rebalanceListner);
        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord<String, String> record : records) {
                        /*System.out.println("Topic:"+ record.topic() + 
                            " Partition:" + record.partition() + 
                            " Offset:" + record.offset() + " Value:"+ record.value());*/
                    // Do some processing and save it to Database
                    rebalanceListner.addOffset(record.topic(), record.partition(), record.offset());
                }
                //consumer.commitSync(rebalanceListner.getCurrentOffsets());
            }
        } catch (Exception ex) {
            System.out.println("Exception.");
            ex.printStackTrace();
        } finally {
            consumer.close();
        }
    }

}  
```