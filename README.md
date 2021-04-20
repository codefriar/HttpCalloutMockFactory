# HttpCalloutMockFactory

## What is this?
This is a bit of code extracted from [Apex Recipes](https://github.com/trailheadapps/apex-recipes). It's labeled @isTest, and can only be used in unit tests. The HttpCalloutMockFactory class enables developers to quickly and effortlessly create objects that conform to the System.HttpCalloutMock interface. Intenteded to be used in conjunction with the `Test.setMock(HttpCalloutMock.class, objVar)` method.

## Install options:
1. [You can install this as a package using this link](https://login.salesforce.com/packaging/installPackage.apexp?p0=04tB0000000QbzOIAS)
2. You can simly copy/paste the contents of force-app/main/default/classes/HttpCalloutMockFactory.cls into your own org.

## Using HttpCalloutMockFactory

Given this bit of Apex code:

```apex
public class ExampleCallout {
    public static List<Account> getAccounts(){
        HttpRequest request = new HttpRequest();
        request.setEndpoint('https://api.example.com/accounts');
        request.setMethod('GET');
        Http protocol = new Http();
        HttpResponse response = protocol.send(request);
        String jsonBody = response.getBody();
        List<Account> accounts = (List<Account>) JSON.deserialize(jsonBody, List<Account>.class);
        return accounts;
    }
}
```

You can construct your mock using the mock factory in two ways:

The first is suitable for mocking a single Http Callout.

```apex
@isTest
private class CalloutRecipes_Tests {
    @isTest
    static void testRawCalloutPositive() { 
        /**
         * This line uses HttpCalloutMockFactory to create a mock objects
         **/
        HttpCalloutMockFactory mock = new HttpCalloutMockFactory(
            200,
            'OK',
            'OK',
            new Map<String, String>()
        );
        /**
         * This next line tells the unit test framework to use your mock object
         **/
        Test.setMock(HttpCalloutMock.class, mock);

        Test.startTest();
        String results = ExampleCallout.getAccounts();
        Test.stopTest();

        // ... and assertions.
    }
}
```

However, if your code makes a sequence of callouts, use the secondary constructor like this:

```apex
@isTest
private class CalloutRecipes_Tests {
    @isTest
    static void testRawCalloutPositive() { 
        /**
         * These next few lines generate a list of HttpResponse objects
         **/
        List<HttpResponse> results = new List<HttpResponse>();
        results.add(
            HttpCalloutMockFactory.generateHttpResponse(200,
            'OK',
            'OK',
            new Map<String, String>())
        )
        results.add(
            HttpCalloutMockFactory.generateHttpResponse(301,
            'OK',
            'MOVED',
            new Map<String, String>())
        )

        /**
         * This line uses HttpCalloutMockFactory to create a mock object capable
         * of handling a sequence of two callouts.
         **/
        HttpCalloutMockFactory mock = new HttpCalloutMockFactory(results);
        /**
         * This next line tells the unit test framework to use your mock object
         **/
        Test.setMock(HttpCalloutMock.class, mock);

        Test.startTest();
        String results = ExampleCallout.MethodThatMakesTwoCallouts();
        Test.stopTest();

        // ... and assertions.
    }
```