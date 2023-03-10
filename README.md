# Amazon Athena UDF Connector OpenSearch Full Text Search Example

This is example Athena UDF Connector to perform full text search on OpenSearch.

This repo is extended from [ahtena-udfs](https://github.com/awslabs/aws-athena-query-federation/blob/master/athena-udfs/src/main/java/com/amazonaws/athena/connectors/udfs/AthenaUDFHandler.java) repo.

## Added UDFs

1. "search": Search OpenSearch

Example query:

```
USING EXTERNAL FUNCTION search(
	host VARCHAR,
	region VARCHAR,
	index VARCHAR,
	keyword VARCHAR,
	limit INTEGER
) RETURNS ARRAY <VARCHAR> LAMBDA 'customudf'
SELECT search(
		'xxx.us-west-2.es.amazonaws.com',
		'us-west-2',
		'index_name',
		'keyword',
		20
	);
```

This would return result [aaa, bbb, ccc, ddd, ...].

2. "search_object": Search OpenSearch and get result as JSON string. This can be parsed to desired data type by Athena.

Example query:

```
USING EXTERNAL FUNCTION search_object(
	host VARCHAR,
	region VARCHAR,
	index VARCHAR,
	keyword VARCHAR,
	limit INTEGER
) RETURNS VARCHAR LAMBDA 'customudf'
WITH dataset AS (
    SELECT CAST(json_extract(search_object(
		'xxx.us-west-2.es.amazonaws.com',
		'us-west-2',
		'index_name',
		'keyword',
		20
	), '$') AS ARRAY(MAP(VARCHAR, VARCHAR))) AS contents
)
SELECT content['title'] AS title, content['exist'] AS exist FROM dataset
CROSS JOIN UNNEST(contents) AS t(content)
```

This would return result

| title | exists  |
| ----- | ------- |
| aaa   |   1     |
| bbb   |   0     |

3. "lucene_search": Search OpenSearch with custom lucene query and get result as JSON string. This can be parsed to desired data type by Athena.

```
USING EXTERNAL FUNCTION lucene_search(
	host VARCHAR,
	region VARCHAR,
	index VARCHAR,
	query VARCHAR,
	fields ARRAY<VARCHAR>,
	limit INTEGER
) RETURNS VARCHAR LAMBDA 'customudf'
WITH dataset AS (
    SELECT CAST(json_extract(lucene_search(
		'xxx.us-west-2.es.amazonaws.com',
		'us-west-2',
		'index_name',
		'title:keyword',
		ARRAY ['title'],
		20
	), '$') AS ARRAY(MAP(VARCHAR, VARCHAR))) AS contents
)
SELECT content['title'] AS title FROM dataset
CROSS JOIN UNNEST(contents) AS t(content)
```

This would return result

| title |
| ----- |
| aaa   |
| bbb   |

### Repositories for AWS built UDFs

1. https://github.com/aws-samples/aws-athena-udfs-textanalytics

## Deploying The Connector

To use this connector in your queries, navigate to AWS Serverless Application Repository and deploy a pre-built version of this connector. Alternatively, you can build and deploy this connector from source follow the below steps or use the more detailed tutorial in the athena-example module:

1. From the athena-federation-sdk dir, run `mvn clean install` if you haven't already.
2. From the athena-udfs dir, run `mvn clean install`.
3. From the athena-udfs dir, run  `../tools/publish.sh S3_BUCKET_NAME athena-udfs REGION` to publish the connector to your private AWS Serverless Application Repository. The S3_BUCKET in the command is where a copy of the connector's code will be stored for Serverless Application Repository to retrieve it. This will allow users with permission to do so, the ability to deploy instances of the connector via 1-Click form. Then navigate to [Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo)
4. Try using your UDF(s) in a query.

## License

This project is licensed under the Apache-2.0 License.