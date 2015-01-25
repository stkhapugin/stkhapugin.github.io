---
layout: post
title: A freeze in Core Data's in-memory store
---

I was unit-testing a class that was synchronizing certain data to CoreData after loading it from API. You need to keep in mind this is a brand-new project. We have two methods in API, one logs you in, the other signs you up and logs in. Both retrieve identical response with user profile and a session token. Hence the tests for the synching methods would be nearly identical, albeit have slightly different parameters.

The unit test I designed works the following way. It creates an in-memory core data stack with two contexts, one of which is then passed to synchronizer object. That object downloads mocked user profile, fetches or creates a proper core data entity, fills out the details and completes. Then I reset the synchronizer's context (to make sure that the returned `NSManagedObjectID` is persistent) and retrieve the freshly fetched profile. I expect it to have all the details filled in correctly. Simple enough.

So I write this test for login method and run it. It passes nicely. I feel happiness inside of me since this is the first test of this in-memory core data tests kind that I ever wrote in production. Then I adapt the parameters in this test in order to test the sign up method. While they do differ slightly, there is nothing that actually matters for this test since API Interactor itself is mocked throughout. Oddly enough, the test *never completes*. Not only that, it never even times out, and memory skyrockets a hundred megs per second. 

After an hour or so, it turns out that the problem is in the core data stack that I never tested, considering it too obvious to even bother. The part that is interesting is, my stack provided a method called `+ (NSManagedObjectContext *) mainContext` — a singleton main-thread context that you can conveniently use in UI-related code. To make things even simpler, I decided to incapsulate its syncrhonization with external changes in the stack as well. Here's what I did:

```objc
- (NSManagedObjectContext*) mainContext
{
    @synchronized(self)
    {
        if (!_mainContext)
        {
            NSPersistentStoreCoordinator *coordinator = [self storeCoordinator];
            if (coordinator != nil)
            {
                _mainContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
                _mainContext.mergePolicy = NSMergeByPropertyStoreTrumpMergePolicy;
                [_mainContext setPersistentStoreCoordinator: coordinator];
                [[[[NSNotificationCenter defaultCenter] rac_addObserverForName:NSManagedObjectContextDidSaveNotification object:nil] deliverOn:[RACScheduler mainThreadScheduler]] subscribeNext:^(NSNotification *notif) {
                    [self.mainContext mergeChangesFromContextDidSaveNotification:notif];
                }];
            }
        }
        
        return _mainContext;
    }
}
```

at the time of writing it felt completely safe to do this. When I was re-reading this piece looking for a bug, it was quite easy to notice: there is a retain cycle (because [RACSignal subscribers retain the channels](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/MemoryManagement.md)). This probably wouldn't have caused problems when you only have one singleton core data stack; however, in each of my two tests I initiated an in-memory stack, and they were stuck there until the tests ended. The first one outlived the test itself because of this retain cycle and received the merge notification, which it tried to merge changes from, which resulted in a lock. 

The question that is open: why did CoreData live-lock inside `mergeChangesFromContextDidSaveNotification:`? Oddly enough, it only happens in in-memory store-backed context merges. [A friend of mine](https://twitter.com/wanderwaltz) suggested that in-memory stores work quite differently from regular ones. I'd say, those are implementation details. My example clearly abuses core data's api by merging two contexts from different stores, so this isn't too surprising that it doesn't work, right?
