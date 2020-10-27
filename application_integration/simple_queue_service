# Simple Queue Service (SQS)
SQS is a hosted queuing service that allows decoupling the components of an application so that they are independent. It provides a generic web services API and it can be accessed by any programming language that the AWS SDK supports. 

## How SQS Works
A queue in SQS consists of a pool of distributed servers which an application's components can push or pull messages from. SQS messages are max 256kB. When SQS receives a message, it is stored redundantly across the pool of SQS servers.

The following scenario describes the lifecycle of an Amazon SQS message in a queue, from creation to deletion.

![](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-message-lifecycle-diagram.png)

When a component starts retrieves a message, the message is marked as invisible until either the message is successfully processed (and is removed from the queue), or if no response is received after the duration of the *visibility timeout* (and becomes visible again, ready to be picked up by another component e.g in an auto-scaling group). 
The default visibility timeout value is **30 seconds**, with a range of **0 seconds to 12 hours**. Messages can remain in the SQS queue for as long as the *retention period*, where the default is **4 days** with a range of **1 minute to 14 days**.

## FIFO Queues
FIFO (First-In-First-Out) queues preserve message order and prevent duplicate messages.

