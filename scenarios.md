# CoCoMo use cases

This document collects a list of CoCoMo use cases.

## ACTORS

PDSs:

- BSky
- ThePiratesStore: an offshore PDS
- EuroPDS: a PDS covered by CoCoMo

AppViews:

- BSky
- Flashes: an AppView that uses CoCoMo

Users:

- Sally: has a repo on BSky PDS
- Jimmy: has a repo on ThePiratesStore
- Sebastian: has a repo on EuroPDS
- Mahoney: a moderator of CoCoMo
- Latife: a Turkish user emigrated in Germany with a EuroPDS account
- Lucy: a UK user using BSky
- Ahmet: a Turkish national living in Turkey using BSky

Moderation:

- CoCoMo: the moderation service in subject
- Ozone: BSky's moderation service

## SCENARIOS

### SCENARIO: user post legal content

    WHEN Sally posts about her cat
    THEN the BSky PDS will accept the write
    AND it will emit a commit event to its subscribers
    WHEN the commit event is received by CoCoMo
    THEN CoCoMo will not mark the content
    AND it will not emit an event to its subscribers

### SCENARIO: user post CSAM content

    WHEN Jimmy posts CSAM content
    THEN the ThePiratesStore will accept the write
    AND it will emit a commit event to its subcribers
    WHEN the commit event is received by CoCoMo
    THEN CoCoMo will mark the commit with a CSAM label
    AND it will emit an event to its subscribers
    AND it will send a notification to the ThePiratesStore
    GIVEN Flashes is subscribing to CoCoMo
    WHEN it will receive the notification of a CSAM content
    THEN it will prevent the content from being shown to its users

### SCENARIO: user reports illegal content

    GIVEN Jimmy post some well-masked CSAM content
    AND the content is not labelled by CoCoMo
    WHEN Sebastian sees the post
    THEN he reports the content through Flashes
    AND Flashes will emit a report event to CoCoMo
    WHEN the report event is received by CoCoMo
    THEN CoCoMo will mark it with an appropriate label
    AND it will emit an event to its subscribers
    AND it will send a notification to ThePiratesStore, Jimmy's PDS
    AND ThePiratesStore will notify Jimmy about the report

### SCENARIO: user appeals a label

    WHEN Jimmy receives a notification from ThePirateStore about a label received for his content
    THEN he will send an appeal request through ThePiratesStore
    AND the appeal request will be sent to CoCoMo
    WHEN the appeal request is received by CoCoMo
    THEN CoCoMo will retain knowledge of the appeal
    AND it will emit an event to its subscribers

### SCENARIO: labelling moderation on CoCoMo

    WHEN Mahoney logins to the CoCoMo mods dashboard
    THEN they will be able to see the list of labels and appeals
    AND they will be able to take further actions
    WHEN Mahoney takes an action on a label or appeal
    THEN CoCoMo will emit an event to its subscribers
    AND it will send a notification to the PDS hosting the repo with the content

### SCENARIO: Sebastians reads old content

    WHEN Sebastian is on a doom scroll on Flashes
    AND it ends up reading content from last year
    AND the content was created before CoCoMo was implemented
    AND Flashes doesn't hold cached data about it
    THEN Flashes will fetch for content + label from CoCoMo
    AND CoCoMo will fetch the content from the PDS
    AND it will apply labelling rules
    AND it will mark with the appropriate label
    AND it will return the content + labels to Flashes
    AND it will emit an event to its subscribers


### SCENARIO: Controversial content can be available or not depending the jurisdictions where the requesting user is located

CONTEXT:

- It is illegal to insult the Turkish president under Turkey's law

*NOTE:*  
*It is unsure whether CoCoMo will support Turkey as a jurisdiction.*  
*There were more obvious scenarios, perhaps too controversial in the current political moment.*  

    GIVEN Latife posts critical content about the Turkish president as a response to a thread under a bsky lexicon
    WHEN Lucy access that thread
    THEN Lucy BSky requests the content to the CoCoMo proxied PDS endpoint
    AND CoCoMo understands the content is legal for Lucy jurisdiction
    AND CoCoMo will serve the content
    AND Lucy will see the content

    WHEN Ahmet access that thread
    THEN Ahmet client requests the content to the CoCoMo proxied PDS endpoint
    AND CoCoMo understands the content is illegal for Ahmet jurisdiction
    AND CoCoMo will not serve the content
    AND Ahmet will see a 403 error with an explanation

### SCENARIO: Rules are updated on CoCoMo

    WHEN CoCoMo updates its labelling rules
    THEN it will DDOS its subscribers with label updates events
    AND it will DDOS PDSs with label updates events
