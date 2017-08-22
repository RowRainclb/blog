---
title: aws在java中使用
date: 2017-08-22 17:01:57
tags:
    - aws
    - dynamodb
    - java
---
一 AWS DynamoDb在java中的使用【建立连接】
``` javascript
accessKey = “xxxxxxx”;
secretKey = “xxxxxxxx”
if (StringUtils.isNotBlank(accessKey) && StringUtils.isNotBlank(secretKey)) {
   logger.debug("accessKey和secretKey有值,不是写在系统配置里的方式");
   bac = new BasicAWSCredentials(accessKey, secretKey);
}
if (bac != null) { 
   client = new AmazonDynamoDBClient(bac);
} else { 
   client = new AmazonDynamoDBClient();
}
String region= “china”;
if (region!= null && region.trim().equals("china")) {
   logger.debug("中国区域");
   regions = Regions.CN_NORTH_1;
   client.withRegion(Region.getRegion(Regions.CN_NORTH_1));
} else {
   logger.debug("us-west-2");
   regions = Regions.US_WEST_2;
   client.withRegion(Region.getRegion(Regions.US_WEST_2));
   //client.setEndpoint("https://dynamodb.us-west-2.amazonaws.com");
}
dynamoDB = new DynamoDB(client);
mapper = new DynamoDBMapper(client);
```

二 AWS DynamoDb在java中的使用【建表】
```javascript
  /**
    * create a table in dynamodb of aws
    * 创建aws表
    * @param tableName          the name of table
    * @param key                分区键(主键)
    * @param keyType
    * @param readCapacityUnits
    * @param writeCapacityUnits
    */
   public void createTable(String tableName, String key, String keyType, Long readCapacityUnits, Long writeCapacityUnits) {
      try {
         ArrayList<AttributeDefinition> attributeDefinitions = new ArrayList<AttributeDefinition>();
         attributeDefinitions.add(new AttributeDefinition()
               .withAttributeName(key)
//             .withAttributeType("N"));
               .withAttributeType(keyType));
         ArrayList<KeySchemaElement> keySchema = new ArrayList<KeySchemaElement>();
         keySchema.add(new KeySchemaElement()
               .withAttributeName(key)
               .withKeyType(KeyType.HASH)); //Partition key
         CreateTableRequest request = new CreateTableRequest()
               .withTableName(tableName)
               .withKeySchema(keySchema)
               .withAttributeDefinitions(attributeDefinitions)
               .withProvisionedThroughput(new ProvisionedThroughput()
               .withReadCapacityUnits(readCapacityUnits)
               .withWriteCapacityUnits(writeCapacityUnits));
         System.out.println("Issuing CreateTable request for " + tableName);
         Table table = dynamoDB.createTable(request);
         System.out.println("Waiting for " + tableName
               + " to be created...this may take a while...");
         table.waitForActive();
      } catch (Exception e) {
         System.err.println("CreateTable request failed for " + tableName);
         System.err.println(e.getMessage());
      }
   }

```
三 AWS DynamoDb在java中的使用【获取表信息】
```javascript
/**
 * Test the infomation of table
 * 获取表的详细信息，描述等属性
 */
public void getTableInformation() {
   System.out.println("Describing " + tableName);
   TableDescription tableDescription = dynamoDB.getTable(tableName).describe();
   System.out.format("Name: %s:\n" + "Status: %s \n"
               + "Provisioned Throughput (read capacity units/sec): %d \n"
               + "Provisioned Throughput (write capacity units/sec): %d \n",
         tableDescription.getTableName(),
         tableDescription.getTableStatus(),
         tableDescription.getProvisionedThroughput().getReadCapacityUnits(),
         tableDescription.getProvisionedThroughput().getWriteCapacityUnits());
}
```
四 AWS DynamoDb在java中的使用【查询所有表】
```javascript
/**
 * List all tables
 * 查询dynamodb 所有的表
 */
public void listMyTables() {
   TableCollection<ListTablesResult> tables = dynamoDB.listTables();
   Iterator<Table> iterator = tables.iterator();
   System.out.println("Listing table names");
   while (iterator.hasNext()) {
      Table table = iterator.next();
      System.out.println(table.getTableName());
   }
}
```
五 AWS DynamoDb在java中的使用【映射查询】
```javascript
/**
 * 查询
 *
 * @param o     表对应的对象
 * @param clazz 表对应的类
 * @return
 */
public <T> List<T> query(T o, Class<T> clazz) {
   DynamoDBQueryExpression<T> queryExpression = new DynamoDBQueryExpression<T>()
         .withHashKeyValues(o);
   List<T> itemList = mapper.query(clazz, queryExpression);
   return itemList;
}


public static void main(String[] args) {
   AWSUtilAPI awsUtilAPI = AWSUtilAPI.getInstanceVersion3();
   DynomadbObject dlc = new DynomadbObject();
   dlc.setBatch("");
   List<DynomadbObject> dlcs= awsUtilAPI.query(dlc,DynomadbObject.class);
}

```
类DynomadbObject
```javascript
package com.test.util;

import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBAttribute;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBRangeKey;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable;
@DynamoDBTable(tableName="td1")
public class DynomadbObject {

      private String batch; //不要用 "#" "()" "空格"这三种字符，amazon识别为非法字符，建议使用 "_" "-"这两种
      private String job;
      private String category;
      private int length;
      private String runname;
      private String status;
      private String progress;
      
      @DynamoDBHashKey(attributeName="batch") 
      public String getBatch() {
         return batch;
      }
      public void setBatch(String batch) {
         this.batch = batch;
      }
      @DynamoDBRangeKey(attributeName="job") 
      public String getJob() {
         return job;
      }
      public void setJob(String job) {
         this.job = job;
      }
      @DynamoDBAttribute(attributeName="category") 
      public String getCategory() {
         return category;
      }
      public void setCategory(String category) {
         this.category = category;
      }
      @DynamoDBAttribute(attributeName="length") 
      public int getLength() {
         return length;
      }
      public void setLength(int length) {
         this.length = length;
      }
      @DynamoDBAttribute(attributeName="runname") 
      public String getRunname() {
         return runname;
      }
      public void setRunname(String runname) {
         this.runname = runname;
      }
      @DynamoDBAttribute(attributeName="status") 
      public String getStatus() {
         return status;
      }
      public void setStatus(String status) {
         this.status = status;
      }
      @DynamoDBAttribute(attributeName="progress") 
      public String getProgress() {
         return progress;
      }
      public void setProgress(String progress) {
         try {
            this.progress = progress;
         }catch (Exception e){
            e.printStackTrace();
         }
      }


}
```
六 根据key查询
 ```javascript
	/**
	 * Return the Item By the primary key and value
	 * 查询某一行的值
	 * @param key		primary key of the table
	 * @param keyValue  the value of primary key
	 * @return
	 */
	public Item query(String key, Object keyValue) {
		logger.debug("进入query()");
		logger.debug("table:"+tableName);
		Table table = dynamoDB.getTable(tableName);
		GetItemSpec spec = null;
		if (keyValue instanceof Number) {
			if (keyValue instanceof Integer) {
				int value = (Integer) keyValue;
				spec = new GetItemSpec().withPrimaryKey(key, keyValue);
			} else if (keyValue instanceof Double) {
				double value = (Double) keyValue;
				spec = new GetItemSpec().withPrimaryKey(key, value);
			}
		} else {
			spec = new GetItemSpec().withPrimaryKey(key, keyValue);
		}
		try {
			Item outcome = table.getItem(spec);
			return outcome;
		} catch (Exception e) {
			System.err.println(e.getMessage());
			e.printStackTrace();
		}
		return null;
	}
```
七 多主键联合查询
```javascript
	/**
	 * Return the Item By the primary key and value
	 * 查询某一行的值
	 * @param key		primary key of the table
	 * @param keyValue  the value of primary key
	 * @return
	 */
	public Item query(String key, Object keyValue) {
		logger.debug("进入query()");
		logger.debug("table:"+tableName);
		Table table = dynamoDB.getTable(tableName);
		GetItemSpec spec = null;
		if (keyValue instanceof Number) {
			if (keyValue instanceof Integer) {
				int value = (Integer) keyValue;
				spec = new GetItemSpec().withPrimaryKey(key, keyValue);
			} else if (keyValue instanceof Double) {
				double value = (Double) keyValue;
				spec = new GetItemSpec().withPrimaryKey(key, value);
			}
		} else {
			spec = new GetItemSpec().withPrimaryKey(key, keyValue);
		}
		try {
			Item outcome = table.getItem(spec);
			return outcome;
		} catch (Exception e) {
			System.err.println(e.getMessage());
			e.printStackTrace();
		}
		return null;
	}
```

八 scan方式查询
  ```javascript
	/**
	 * scan方式查询
	 * scan方式查询dynamodb 表的数据
	 *
	 * 为结果分页

	 DynamoDB 会对 Query 和 Scan 操作的结果进行分页。分页后，Query 和 Scan 结果会划分到不同的页；应用程序可以先处理第一页结果，然后处理第二页结果，以此类推。从 Query 或 Scan 操作返回的数据限制为 1 MB；这意味着，如果结果集超出数据的 1 MB，您将需要执行另一个 Query 或 Scan 操作来检索数据的下一个 1 MB。

	 如果您查询或扫描的特定属性的匹配值总数超过 1 MB 个数据，则需要再执行一次 Query 或 Scan 请求以获得后续 1 MB 个数据。为此，请从上一个请求获取 LastEvaluatedKey 值，将该值用作下一个请求中的 ExclusiveStartKey。利用此方法，您能够以 1 MB 为增量渐进式查询或扫描新数据。

	 在处理完来自 Query 或 Scan 的整个结果集后，LastEvaluatedKey 是 null。这表明，此结果集是完整的（即该操作处理的是“最后一页”数据）。

	 如果 LastEvaluatedKey 是除 null 以外的任何值，这an不一定意味着结果集中具有更多数据。了解您何时达到了结果集末尾的唯一方式是当 LastEvaluatedKey 是 null 时

	 * @param tableName	表名
	 * @param filterColumn	过滤列名
	 * @param filterValue	过滤列值
	 * @param columns	查询列名，逗号分割
	 * @param reservedKeyWordColumn 列名是保留关键字的，以逗号分割
	 */
	public List<Map<String,String>>  scan(String tableName,String filterColumn,String filterValue,String columns,String reservedKeyWordColumn){
		List<Map<String,String>> list = new ArrayList<>();
		Map<String, AttributeValue> expressionAttributeValues =
				new HashMap<>();
		expressionAttributeValues.put(":val", new AttributeValue().withS(filterValue));
		Map<String, String> expressionAttributeNames =
				new HashMap<>();
		StringBuffer rColumns = new StringBuffer("");
		if(StringUtils.isNotBlank(reservedKeyWordColumn)){
			String[] rKeys = reservedKeyWordColumn.split(",");
			for(String rKey:rKeys){
				expressionAttributeNames.put("#"+rKey,rKey);
				rColumns.append(",");
				rColumns.append("#"+rKey);
			}
		}
		String projections = columns + rColumns.toString();
		ScanRequest scanRequest = new ScanRequest()
				.withTableName(tableName)
				.withFilterExpression(filterColumn+" = :val")
				.withProjectionExpression(projections)
				.withExpressionAttributeValues(expressionAttributeValues)
				.withLimit(50);
		if(expressionAttributeNames.size() > 0){
			scanRequest.withExpressionAttributeNames(expressionAttributeNames);
		}
		ScanResult result = client.scan(scanRequest);
		String[] columnNames = columns.split(",");
		String[] rColumnNames = reservedKeyWordColumn.split(",");
		List<Map<String,String>>  list1 = parseToList(result,columnNames,rColumnNames);
		list.addAll(list1);
		while (result.getLastEvaluatedKey()!=null){
			scanRequest.setExclusiveStartKey(result.getLastEvaluatedKey());
			result = client.scan(scanRequest);
			List<Map<String,String>>  list2 = parseToList(result,columnNames,rColumnNames);
			list.addAll(list2);
		}
		return list;
	}

```
 九 根据key查询某个字段的值
```javascript
	/**
	 * 根据key查询某个字段的值
	 * 适合取某个字段的值
	 *
	 * @param key       key名
	 * @param keyValue  key值
	 * @param param     字段名
	 * @return 字段值
	 */
	public String getValuesByParam(String key, Object keyValue, String param) {
		Item item = query(key, keyValue);
		if (item == null) {
			return null;
		}
		String value = item.getString(param);
		return value;
	}
```
十 推送aws(插入记录)
```javascript
   /**
	 * 推送AWS
	 *
	 * @param path  batch.lst绝对路径
	 * @param batch 封装的Batch对象，需要参数：batch[生风任务编号],owner[生风所有者 默认为：k2data/当前登陆用户],version[ 程序版本 当前版 ：0.2],stage[默认位：wind_gen]
	 */
	public boolean insertItem(String path, Batch batch) {
		logger.debug("准备插入batch表:"+tableName);

		List<String> list = getFilePaths(path);
		checkAwsIllegalToken(list);
		String task_list = makeCode(list, batch.getStage());
		int calculationTime = evalCalculationTime(path, batch.getStage());
		batch.setCalculation_time(calculationTime);
		Date date = new Date();
		DateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String time = format.format(date);//time就是当前时间
		batch.setCreated(time);
		batch.setTotalDlcCount(list.size());
			/*batch.setOwner(batch.getOwner());
			batch.setVersion(batch.getVersion());*/
		batch.setTask_list(task_list);

		List<Item> items = new ArrayList<Item>();
		Item item = new Item()
				.withPrimaryKey("batch", batch.getBatch())
				.withString("created", batch.getCreated())
				.withNumber("total_dlc_count", batch.getTotalDlcCount())
				.withNumber("calculation_time", batch.getCalculation_time())
				.withString("owner", batch.getOwner())
				.withString("version", batch.getVersion())
//				.withString("stage", batch.getStage())
				.withString("status", batch.getStatus())
				.withString("task_list", batch.getTask_list());
		items.add(item);
		logger.debug("createItems  start...");
		return createItems(tableName, items);
	}

	/**
	 * 创建多个项目
	 *相当于sql中的添加多条记录
	 * @param tableName 表名
	 * @param list      列组合的item
	 */
	public boolean createItems(String tableName, List<Item> list) {
		Table table = dynamoDB.getTable(tableName);
		try {
			for (int i = 0; i < list.size(); i++) {
				Item item = list.get(i);
				PutItemOutcome outcome = table.putItem(item);
				logger.debug("AWS 生风任务推送成功:\n" + outcome.getPutItemResult());
			}
			return true;
		} catch (Exception e) {
			logger.debug("AWS 生风任务推送失败.");
			logger.debug(e.getMessage());
		}
		return false;
	}

	/**
	 * 创建单个项目
	 *
	 * @param tableName 表名
	 * @param item      列组合的item
	 */
	public void createItem(String tableName, Item item) {
		Table table = dynamoDB.getTable(tableName);
		try {
			PutItemOutcome outcome = table.putItem(item);
			System.out.println("PutItem succeeded:\n" + outcome.getPutItemResult());
		} catch (Exception e) {
			System.err.println("Create items failed.");
			System.err.println(e.getMessage());
		}
	}

```

