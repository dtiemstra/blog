---
title: 'Prevent valid messages from being rejected by AWS Lambda'
summary: 'By default, a batch of messages being processed by a Lambda function through SQS polling would either be completely successful, in which case the records would be deleted from the SQS queue, or would completely fail, and the records would be kept on the queue to be reprocessed. The Partial Batch Response feature an SQS queue will only retain those records which could not be successfully processed, preventing valid messages from being rejected.'
date: 2023-05-02
author: 'Diederik Tiemstra'
tags: ['AWS Lambda', '.NET']
draft: false
---

AWS Lambda is a powerful tool that allows developers to build scalable and efficient serverless applications. One of the key features of Lambda is its ability to process batches of messages efficiently. However, when working with batches, the default behavior is to reject all messages in the batch when processing of a single message fails. This can lead to valid messages being transferred to a Dead Letter Queue. Fortunately you can use the partial batch response to prevent this from happening.

## The scenario

To understand partial batch response, let's consider an example. Suppose you have an Amazon Simple Queue Service (SQS) queue with a dead letter queue and a Lambda function that processes a batch of messages from that SQS queue. The Lambda function processes each message sequentially and performs some logic. Now, let's say that one of the messages in the batch fails during processing due to an error. Without partial batch response, the entire batch would be rejected, and all of the messages would end up in the configured dead letter queue.

## The Solution

The solution is to implement the partial batch response feature. With partial batch response, the Lambda function can indicate which messages in the batch were not processed successfully. The Lambda function can return a response that includes a list of message IDs for the messages that were not processed successfully. The response can be sent back to SQS, and SQS can then remove the successfully processed messages from the queue and leave the failed messages in the queue for further processing or investigation.

To enable this feature you must first set the `reportItemFailures` on the eventsource of the lambda

```
#example for setting the reportBatchItemFailures using aws cdk
const myEventSource = new SqsEventSource(myQueue, {
	reportBatchItemFailures: true,
});
myLambda.addEventSource(myEventSource);
```

Secondly, make sure your lambda function returns a `SQSBatchResponse` object. With this object you have fine grained control on which items are rejected by your function.

```
public async Task<SQSBatchResponse> ProcessMessages(SQSEvent sqsEvent)
{
	SQSBatchResponse response = new();
	foreach(var message in sqsEvent.Records)
	{
		try
		{
			//perform some logic here
		}
		catch(Exception e)
		{
			response.BatchItemFailures.Add(new SQSBatchResponse.BatchItemFailure { ItemIdentifier = message.MessageId });
		}
	}
	return response;
}
```

## Not only SQS

Setting the ReportItemFailures is not only limited to AWS SQS as the datasource for your Lambda function. When using Kinesis or DynamoDB Streams the ReportItemFailures can also be set.
