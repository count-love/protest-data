# Introduction
This is an annotated dataset of news article URLs that potentially contain reports about protest events
in the United States between January 2017 and September 2019. These URLs were collected (1) automatically,
via a crawler that searched local news and metro section homepages for the protest stem words “rally,”
“march,” “protest,” or “demonstration;” and (2) manually, via web searches for known missing reports
(such as missing news articles about planned protests as part of a national march).

The URLs in this dataset may point to articles that are no longer available for a variety of reasons,
including publishers moving content behind a paywall or deleting articles older than a specific
expiration. Where possible, when multiple articles report about the same protest, all article references
are listed in the corresponding entry in the `events` hash.

# Data Organization
The dataset is a single JSON file with the following structure:

```json
{
    "tags": {
        "category": array of category tags,
        "position": array of position tags,
        "detail": array of detail tags
    },
        
    "articles": [{
        "article_id": unique article ID (integer),
        "test_set": indicates whether the article is in the test set (boolean; false for training set, true for test set),
        "date_crawled": date that article was read ("YYYY-MM-DD"),
        "origin": denotes whether article was discovered by the crawler ("crawler" | "manual")
        "events": [{
            "event_id": event_id of the event that this article describes,
            "future": whether or not the article reference to this event is in the future (boolean; false for past reference, true for future reference)
        }]
    }],
    
    "events": [{
        "event_id": unique event ID (integer),
        "event_date": date of protest event ("YYYY-MM-DD"),
        "event_location": location of protest event (string),
        "event_attendees": lowest, most specific number of attendees for this event, if reported (integer | null),
        "event_tags": semicolon separated string of category, position, and detail tags,
        "event_primary_url": the primary source of event details,
        "articles": [{
            "article_id": article_id of an article that references this event,
            "future": whether or not this article's reference of the event is in the future (boolean; false for past reference, true for future reference)
        }]    
    }]
}
```

## Tags
The “tags” hash contains three different string arrays that enumerate all of the tags in the protest taxonomy. Category tags, such as “Civil Rights” or “Immigration,” broadly describe protest topics. Position tags, which start with “For,” “Against,” “Pro,” or “Anti,” indicate what protestors are advocating for or against. Detail tags indicate more information, such as the name of a corresponding national event (e.g., “March for Our Lives”) or a recurring detail (e.g., “Police”). While tags are not mutually exclusive, in practice some tags are semantically opposite and never appear together for the same event. All events have at least one category tag and one position tag.

## Articles
The `articles` array contains every article in this dataset. Each array entry is a hash that contains metainformation about an article, including its unique `article_id`, the date that the article was crawled (or published for articles prior to February 19, 2017, the date that this project started), and an `origin` field denoting whether an article was automatically found by the crawler or manually added. 

Each article entry also contains an `events` array, and each element in the `events` array represents a protest event found in the article. Individual events have a unique `event_id` that matches the `event_id` field in the top level `events` array. Because this dataset also includes negative examples where the titles of articles contain protest stem words but the article itself is not about a protest, some events arrays are empty. Each event also contains a `future` boolean, where `0` indicates that the article describes a protest that happened in the past, and a `1` indicates that the article describes a protest that will happen in the future.

## Events
The `events` array contains every protest event in this dataset. Each element is a hash that contains metainformation about a protest, such as the reasons for protest or the number of attendees at a protest. The reasons for protest are listed as a semicolon separated string and populated with values from the top level `tags` hash. The `event_primary_url` field indicates the primary source of information for an event because multiple articles can report about the same protest event. All `article_id` entries match the `article_id` field in the top level `articles` array.

In the `event_attendees` field, for events that have multiple reference articles, we record the smallest, most precise attendee count with subjective judgements for source reliability. We record crowd estimates of “a dozen” as 10, “dozens” as 20, “hundreds” as 100, and so forth. We also record “several dozen” as 20, “several hundred” as 200, etc. Events with `0` attendees represent activities such as banner drops, while events without a reported count have a `null` value.

# Example Data Excerpt
The following is a simple data excerpt from the full dataset to illustrate the relationship between tags, articles, and events in the full dataset.

```json
{
    "tags": {
        "category": [
            "Civil Rights"
        ],
        "position": [
            "For racial justice",
            "Against white supremacy",
        ],
        "detail": [
            "Charlottesville"
        ]
    },
    
    "articles": [
        {
            "article_id": 36922,
            "test_set": true,
            "date_crawled": "2017-08-13",
            "origin": "crawler",
            "url": "http://www.baltimoresun.com/entertainment/dining/baltimore-diner-blog/bs-md-ci-charlottesville-rally-20170812-story.html",
            "events": [
                {
                    "event_id": 7189,
                    "future": false
                }
            ]
        },
        {
            "article_id": 57685,
            "test_set": false,
            "date_crawled": "2017-10-08",
            "origin": "crawler",
            "url": "http://baltimore.cbslocal.com/2017/10/08/kkk-leader-charlottesville-gun-charge/",
            "events": [
                {
                    "event_id": 7189,
                    "future": false
                }
            ]
        },        
    ],
    
    "events": [
        {
            "event_id": 7189,
            "event_date": "2017-08-12",
            "event_location": "Baltimore, MD",
            "event_attendees": 40,
            "event_tags": "Civil Rights; For racial justice; Against white supremacy; Charlottesville",
            "event_primary_url": "http://www.baltimoresun.com/entertainment/dining/baltimore-diner-blog/bs-md-ci-charlottesville-rally-20170812-story.html",
            "articles": [
                {
                    "article_id": 36922,
                    "future": false
                },
                {
                    "article_id": 57685,
                    "future": false
                }
            ]
        }
    ]
}
```