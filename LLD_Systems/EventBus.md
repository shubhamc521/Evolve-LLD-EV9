
There are multiple microservices.

Service A is sending the msg to Service B but Service B is not available.
If not available, I wil retry for a while and I will stop. Message will get lost.


Service B: -----Depositing the money.
Service A: -----Withdraw (ATM)

If Service B is coming first and then service A but somehow Service A req handled first.
Inssfficient Bal.

Order should mentained.

[ Queue - All the msg, will be stored in this queue and proccesed in sequential manner.]

[YT Channel 1]
Publisher A:------                                  -----Subscriber X. - YT Channel 1
                  ------[ EventBus YT1, YT2]  ------     Subscriber y. - YT Channel 2
Publisher B:------                                  ---- Subscriber Z. - YT Channel 1, 2
[YT Channel 2]


Event Bus can PUSH the message to the interested Subcriber 
OR
Subscribers can PULL the message from Event Bus.

Publisher == Producer
Subcriber == Consumer

Key Concepts:
Events = Events are messages or notif about a change or action that has happened. [New Video Uploaded (Update/Action)]
                                                                                  [User Updates / LinkedinPost]

Publisher-Subscriber Model:
    - Publisher: These are event producers. [YT Channel 1]
    - Subscribers: They listen for specific events. [ Subscriber 1 is listening to YT Channel 1's update]

Benfits of Event Bus;
    - Loose Coupling: Publishers and Subscriber are indenpendent. Its easier to maintain and scale.
    - Asynchronous Communication: Publisher can publish the msg and subcriber can process the msg according to their own pace. 
                                  Its like broadcasting invitation to 100 people, Without waiting for ack.

    - Event Driven - PaymentProccessed -----> OrderPlaced ------> OrderShipped
                     What happens if I ship the order before payment? Is it the right thing?
                     Event should happen in proper order.

Partition Events By Topic:
    Bosscoder Academy:    Array                        #DSA #Code                Person1 subcribed to #DSA
    Coders Academy:       LinkedList, SystemDesign.    #DSA #LinkedList          Person2 subcriber to DSA and HLD both.
                                       #HLD

                    Event BUS:

                            DSA  :  ----------------------Array, LinkedList-----           Person1 Interested in DSA thing.
                            HLD  :  ----------------------SystemDesign----------           Person2 Interested in DSA and HLD both.
                            Code :  ----------------------Array, LinkedList----- 

                    These DSA Bus and HLD bus are called Topics.

                    One Subcriber can subcribe to multiple topics.
                    One Publisher can publish to multiple topics.

                    Single Topic: Event will be published to the queue matching the event type.
                    Multiple Topic: Some event will be associated with multiple topics, they will be published to all those topics.

Idempotency:

- Ensures event is proccesed once. Even if it is published to multiple times.

Deposit money in bank, --Multiple request are published by the Deposit Microservice in the queue.
                        It will get proccessed multiple times.
                        THats's wrong?

                        Mechanism:
                         -UniqueEventID
                         -Deduplication Logic: Subcribers should be able to check if the event is already proccessed or not.

                         If order event is published twice, the payment serive should only charge the customer once.

Handling Failures and Retrying events:

- Failure handling: If Event Bus failes to get the ACK from subcriber, then Event Bus should RETRY.
- Retries:
        Exponential backoff: Retry after increasingly longer interval. [You enter wrong password, your phone waits for 10sec, next wrong   try it waits for 20 sec and it goes on.]

        Max Retry Limit: A limit on Retry.

PUSH/PULL Machanism:

    PUSH : Event bus should be able to PUSH event to subcribers.
    PULL : Subscriber should be able to PULL events from the Event BUS.

    PULL after ID/Time:
    Subcriber can requesting all the event from mentioned ID/TimeStamp.

Ordering Event Proccessing:
    
    Sequencial Delivery: Events should be delivered in the order they are published. One at a time. [Logical Clocks like Vector Clocks]
    Transactional Proccessing: If event A fails, Event B should be proccesed.

    Ex: PaymentProccess fails then OrderPlaced event should not be proccessed.
        Ordershipped event should not happen before OrderPlaced event is proccessed successfully.

Bus Mechanics:
    Causal Ordering:
        If event A heepens before event B, then event A should be proccessed before B.

        Threads for Causal Ordering:
        We will signal all the related events to the same thread.

    Coarse Grained Ordering:
        Topic Level Ordering: All the event in that specific topic should be proccessed in the order they are received.
        Hashing for Topic-Level Ordering: h(TopicID) % Size. This ensures the events from same topic goes to same thread.

    Ordering Publishers:
        Multiple Publishers can send events to same topic. All event belonging to the topic need to ordered correctly.
        Thread will be assinged to publisher based on Topic Hash.

Dead Letter Queue:
    If an event cannot be suucessfully proccesed by subcriber after RETRY, the event will be pushed to DLQ.
    Use:
    Sending the failedEvent again.
    Capture the failure logs.
    Prevent the blocking of event proccesing.







Design A EventBus:

- Requirement & Architecture
- Data Models
- Core EventBus
- Keyed Executor - (Thread Mechanism)
- Retry Mechanism
- Dead Letter Queue
- Adv Indexing
- Testing 
- Trade-Off

1. Requirements:

Functional:
- Publish Events to Topics
- Subcribe (PUSH/PULL)
- Retry with BackOFF
- Dead Letter Queue.
- Event Replay
- Direct Lookup

Non Functional:
- Thread Safe
- Ordered with topics
- O(1) Operations
- At least one delivery


Architecture:
- Publisher ----[publish]------EventBus------[route]------Topics-----[Push]----PushSub ----[Fails]----Retry---[max]----DLQ
                                                                -----[Pull]----PullSub

2. Data Models:

```java
public class Event {
    private final EventID id; //identify unique event
    private final String name; //Type of category of event eg. Order.created, user.registered
    private final Map<String, String> attributes; // Key value pair for event details. {OrderId: "123", amount:500}
    private final Timestamp timestamp; // When this event was created.

    public Event(EventID id, String name, Map<String, String> attributes, Timestamp timestamp) {
        this.id = id;
        this.name = name;
        this.attributes = attributes;
        this.timestamp = timestamp;
    }

    public EventID getId() {
        return id;
    }

    public Timestamp getTimestamp() {
        return timestamp;
    }

    public String getName() {
        return name;
    }

    public Map<String, String> getAttributes() {
        return attributes;
    }
    //Convert event details to readable format format logging.
    @Override
    public String toString() {
        return "Event{" +
                "id=" + id +
                '}';
    }
}
```

```java
public class Subscription {
    private final Topic topicId; // Orders, payment, DSA, SystemDesign
    private final EntityID subscriberId; //Unique ID for Service/Appplication. Ex: EmailService1213
    private final SubscriptionType type; // PUSH or PULL
    private final Function<Event, Boolean> precondition; // Condition based delivery
    private final Function<Event, Void> handler; // what to do when event arrives

    public Subscription(final Topic topicId,
            final EntityID subscriberId,
            final SubscriptionType type,
            final Function<Event, Boolean> precondition,
            final Function<Event, Void> handler) {
        this.topicId = topicId;
        this.subscriberId = subscriberId;
        this.type = type;
        this.precondition = precondition;
        this.handler = handler;
    }

    public Function<Event, Void> handler() {
        return handler;
    }

    public SubscriptionType getType() {
        return type;
    }

    public Topic getTopicId() {
        return topicId;
    }

    public EntityID getSubscriberId() {
        return subscriberId;
    }
}
```

2. Supporting Classes:

```java
// Index.java

//Bookmark the you use while reading a book
package models;

public class Index {
    int val; // the POS no in a queue.

    public Index(int val) {
        this.val = val;
    }

    public Index increment() {
        return new Index(val + 1);
    }

    public int getVal() {
        return val;
    }
}

// FailureEvent.java
package models;

import java.util.UUID;

public class FailureEvent extends Event {
    private final Throwable error;
    private final Timestamp timestamp;

    public FailureEvent(Event event, Throwable error, Timestamp timestamp) {
        super(new EventID(UUID.randomUUID().toString()), event.getName(), event.getAttributes(), event.getTimestamp());
        this.error = error;
        this.timestamp = timestamp;
    }
}
```

```java
// It is a thread Pool that ensures ordering.

//Problem: Regular thread pool can proccess event out of order.
//Sol: Same ket always goes to same thead. (Single thread per key)

// KeyedExecutor.java
package utils;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CompletionStage;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;
import java.util.function.Supplier;

public class KeyedExecutor {
    // Array of single threaded Executors
    private final Executor[] executor;

    // Creates pool of single threaded executors.
    public KeyedExecutor(final int threads) {
        executor = new Executor[threads];
        // Create each Executor
        for (int i = 0; i < threads; i++) {
            executor[i] = Executors.newSingleThreadExecutor();
        }
    }
    // Submit a task with a key
    // task is code to run.
    public CompletionStage<Void> submit(final String id, final Runnable task) {
        // hash the id to get a number
        // maaping the hash using array index using modulo
        // Submit the task to selected executor
        return CompletableFuture.runAsync(task, executor[id.hashCode() % executor.length]);
    }
    //get - submit the task that return a value
    public <T> CompletionStage<T> get(final String id, final Supplier<T> task) {
        // 
        return CompletableFuture.supplyAsync(task, executor[id.hashCode() % executor.length]);
    }
}
```
Operation  Key           
publish.   topic.getname() Sequential per topic
poll       topic + subcriber
push       topic = subscriber 

```java
// RetryAlgorithm.java
package retry;

import exception.RetryAbleException;
import exception.RetryLimitExceededException;

import java.util.function.Function;

public abstract class RetryAlgorithm<PARAMETER, RESULT> {

    private final int maxAttempts;
    private final Function<Integer, Long> retryTimeCalculator;

    public RetryAlgorithm(final int maxAttempts,
            final Function<Integer, Long> retryTimeCalculator) {
        this.maxAttempts = maxAttempts;
        this.retryTimeCalculator = retryTimeCalculator;
    }

    public RESULT attempt(Function<PARAMETER, RESULT> task, PARAMETER parameter, int attempts) {
        try {
            return task.apply(parameter);
        } catch (Exception e) {
            if (e.getCause() instanceof RetryAbleException) {
                if (attempts == maxAttempts) {
                    throw new RetryLimitExceededException();
                } else {
                    final RESULT result = attempt(task, parameter, attempts + 1);
                    try {
                        Thread.sleep(retryTimeCalculator.apply(attempts));
                        return result;
                    } catch (InterruptedException interrupt) {
                        throw new RuntimeException(interrupt);
                    }
                }
            } else {
                throw new RuntimeException(e);
            }
        }
    }
}
```

```java

// EventBus.java
import exception.RetryLimitExceededException;
import models.*;
import retry.RetryAlgorithm;
import utils.KeyedExecutor;
import utils.Timer;

import java.util.*;
import java.util.concurrent.CompletionStage;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentSkipListMap;
import java.util.concurrent.CopyOnWriteArrayList;

public class EventBus {
    private final KeyedExecutor executor; //thread management
    private final Map<Topic, List<Event>> buses; // Topic('order') ---> [Event1, Event2 .....]
    private final Map<Topic, Set<Subscription>> subscriptions; // Topic('order' ----> EmailService, ShippingServie)
    private final Map<Topic, Map<EntityID, Index>> subscriberIndexes; //Topic('Order') -->Subscriber1--->Index2
    private final Map<Topic, Map<EventID, Index>> eventIndex; // Event with Index
    private final Map<Topic, ConcurrentSkipListMap<Timestamp, Index>> timestampIndex; // Topic(Order)---> Timestamp100 - Index(0)
    private final RetryAlgorithm<Event, Void> retryAlgorithm; // How to handled failed proccess
    private final EventBus deadLetterQueue;
    private final Timer timer;

    public EventBus(final int threads,
            final RetryAlgorithm<Event, Void> retryAlgorithm,
            final EventBus deadLetterQueue,
            final Timer timer) {
        this.retryAlgorithm = retryAlgorithm;
        this.deadLetterQueue = deadLetterQueue;
        this.timer = timer;
        this.buses = new ConcurrentHashMap<>();
        this.executor = new KeyedExecutor(threads);
        this.subscriptions = new ConcurrentHashMap<>();
        this.subscriberIndexes = new ConcurrentHashMap<>();
        this.timestampIndex = new ConcurrentHashMap<>();
        this.eventIndex = new ConcurrentHashMap<>();
    }

    public CompletionStage<Void> publish(Topic topic, Event event) {
        return executor.submit(topic.getName(), () -> addEventToBus(topic, event));
    }

    //Adding the event to the Bus and Notifiying the consumers/Subcribers.
    private void addEventToBus(Topic topic, Event event) {
        //Get the pos for new event
        final Index currentIndex = new Index(buses.get(topic).size());
        // update the timestamp
        timestampIndex.get(topic).put(event.getTimestamp(), currentIndex);
        //Updating the event id index 
        eventIndex.get(topic).put(event.getId(), currentIndex);
        // Add event to storage
        buses.get(topic).add(event);
        // Notify the PUSH Subcribers // iMplementation of Observer Design pattern.
        subscriptions.getOrDefault(topic, Collections.newSetFromMap(new ConcurrentHashMap<>()))
                .stream()
                .filter(subscription -> SubscriptionType.PUSH.equals(subscription.getType()))
                .forEach(subscription -> push(event, subscription));
    }

    // Get the next event for that subscriber
    public CompletionStage<Event> poll(Topic topic, EntityID subscriber) {
        return executor.get(topic.getName() + subscriber.getId(), () -> {
            final Index index = subscriberIndexes.get(topic).get(subscriber);
            try {
                final Event event = buses.get(topic).get(index.getVal());
                subscriberIndexes.get(topic).put(subscriber, index.increment());
                return event;
            } catch (IndexOutOfBoundsException exception) {
                return null;
            }
        });
    }
//Deliver the Event to the Subscriber
    private void push(Event event, Subscription subscription) {
        executor.submit(subscription.getTopicId().getName() + subscription.getSubscriberId(),
                () -> {
                    try {
                        retryAlgorithm.attempt(subscription.handler(), event, 0);
                    } catch (RetryLimitExceededException e) {
                        if (deadLetterQueue != null) {
                            deadLetterQueue.publish(subscription.getTopicId(),
                                    new FailureEvent(event, e, timer.getTime()));
                        } else {
                            e.printStackTrace();
                        }
                    }
                });
    }
// Whenever new topic is comming, Create Data Stucture topic
    public void registerTopic(Topic topic) {
        buses.put(topic, new CopyOnWriteArrayList<>());
        subscriptions.put(topic, Collections.newSetFromMap(new ConcurrentHashMap<>()));
        subscriberIndexes.put(topic, new ConcurrentHashMap<>());
        timestampIndex.put(topic, new ConcurrentSkipListMap<>());
        eventIndex.put(topic, new ConcurrentHashMap<>());
    }
//
    public CompletionStage<Void> subscribe(final Subscription subscription) {
        return executor.submit(subscription.getTopicId().getName(), () -> {
            final Topic topicId = subscription.getTopicId();
            subscriptions.get(topicId).add(subscription);
            subscriberIndexes.get(topicId).put(subscription.getSubscriberId(),
                    new Index(buses.get(topicId).size()));
        });
    }
// Set the subscriber position after the timestamp
    public CompletionStage<Void> setIndexAfterTimestamp(Topic topic, EntityID subscriber, Timestamp timestamp) {
        return executor.submit(topic.getName() + subscriber.getId(), () -> {
            final Map.Entry<Timestamp, Index> entry = timestampIndex.get(topic).higherEntry(timestamp);
            if (entry == null) {
                subscriberIndexes.get(topic).put(subscriber, new Index(buses.get(topic).size()));
            } else {
                final Index indexLessThanEquals = entry.getValue();
                subscriberIndexes.get(topic).put(subscriber, indexLessThanEquals);
            }
        });
    }
//
    public CompletionStage<Void> setIndexAfterEvent(Topic topic, EntityID subscriber, EventID eventId) {
        return executor.submit(topic.getName() + subscriber.getId(), () -> {
            final Index eIndex = eventIndex.get(topic).get(eventId);
            subscriberIndexes.get(topic).put(subscriber, eIndex.increment());
        });
    }
//Event details
    public CompletionStage<Event> getEvent(Topic topic, EventID eventId) {
        return executor.get(topic.getName(), () -> {
            Index index = eventIndex.get(topic).get(eventId);
            return buses.get(topic).get(index.getVal());
        });
    }
}

```

























