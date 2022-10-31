---
title: Build OData Queries with the SAP Cloud SDK's Virtual Data Model QA Green upd 222
description: Build OData queries with the SAP Cloud SDK's virtual data model to build an address manager application.
auto_validation: true
author_name: Test TesT
author_profile: https://github.com/mervey45
creator_name: Test Test T.
time: 30
tags: [ tutorial>intermediate, topic>javascript, products>sap-business-technology-platform, topic>odata]
primary_tag: products>open-Connectors
---

## Prerequisites
 - Have `Node.js` and `npm` [installed on your machine](s4sdkjs-prerequisites).
 - Have access to an SAP S/4HANA Cloud system or the [SAP API Business Hub Sandbox](https://api.sap.com/getting-started), or use the [Business Partner Mock Service](https://sap.github.io/cloud-s4-sdk-book/pages/mock-odata.html).
 - Basic knowledge of OData is recommended, but not required.

## Details

> ### We migrate tutorials to our [documentation](https://sap.github.io/cloud-sdk/)
> This tutorial is not actively maintained and might be partially outdated.
> Always up-to-date documentation is published on our [documentation portal](https://sap.github.io/cloud-sdk/).
> We will provide a link to the updated version of this tutorial as soon as we release it.
> In this tutorial, version 1 of the SAP Cloud SDK for TypeScript/JavaScript is used.

### You will learn
  - What the OData Virtual Data Model for SAP S/4HANA Cloud is
  - How to use the Virtual Data Model to query the business partner service
  - How to expose the business partners in a `NestJS` application

The goal of this tutorial group is to show you how to implement a JavaScript application that allows you to manage the addresses of business partners. This application will be using `NestJS` and the SAP Cloud SDK for JavaScript. In this tutorial, we introduce the SAP Cloud SDK's OData Virtual Data Model and teach you how to use it to query business partners from an SAP S/4HANA Cloud system.

---

[ACCORDION-BEGIN [Step 1: ](Add an API endpoint)]

A `NestJS` application is based on three main entities: `Service`, `Controller` and `Module`.

In the scaffold a set of these entities has already been created: `app.service.ts`, `app.controller.ts` and `app.module.ts`. We will create a new set in order to query business partners from a S/4HANA Cloud system. For details on the concepts have a look at the [Nest tutorials](https://docs.nestjs.com/first-steps). Start with a `business-partner.service.ts` file, which will contain the implementation of the query:

```JavaScript / TypeScript[3,5-7,11]
import { Injectable } from '@nestjs/common';

@Injectable()
export class BusinessPartnerService {
  public dummyText = 'Not yet implemented.';

  getAllBusinessPartners(): string {
    return this.dummyText;
  }
}
```

Then create a `business-partner.controller.ts` which takes care of directing incoming requests to the needed implementation:

```JavaScript / TypeScript
import { Controller, Get } from '@nestjs/common';
import { BusinessPartnerService } from './business-partner.service';

@Controller('business-partners')
export class BusinessPartnerController {
  constructor(private readonly businessPartnerService: BusinessPartnerService) {}

  @Get()
  getAllBusinessPartners(): string {
    return this.businessPartnerService.dummyText;
  }
}
```

As a last step we have to register the controller and service. In order to keep the example brief, just add the controller and service to the `app.module.ts` which is the root module of the Nest application:

```JavaScript / TypeScript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { BusinessPartnerController } from './business-partner.controller';
import { BusinessPartnerService } from './business-partner.service';

@Module({
  imports: [],
  controllers: [AppController,BusinessPartnerController],
  providers: [AppService,BusinessPartnerService],
})
export class AppModule {}
```

Now you can start your application using:
```Shell
npm start
```
When the server is running, you should see a message like: `[NestApplication] Nest application successfully started` and some mappings related to the listed controllers. Open the URL `http://localhost:3000/business-partners` and you should see `Not yet implemented.`

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 4: ](Query the business partner service)]

Every request you can do with the VDM follows a common pattern. You start with the entity that you want to perform a request on, in this case `BusinessPartner`. To build a request, call the `requestBuilder` function.
Next, you can select which request to build. OData, as RESTful API protocol, follows the CRUD model: Create, Read, Update, Delete. For reading, we differentiate between querying the service, where you get all available entities if you don't specifically restrict the result set, and retrieving a specific entity by its key.
The respective functions are called `getAll` and `getByKey`. For the remaining requests, the functions are simply called `create`, `update` and `delete`. When you type `BusinessPartner.requestBuilder().`, your IDE should show which operations are available on the respective entity. For example, it is not possible to delete a business partner via the business partner service. Therefore, the VDM will not offer a `delete` function.

Following this pattern, update your service as shown below (mind the updated `import`):

```JavaScript / TypeScript
import { Injectable } from '@nestjs/common';
import { BusinessPartner } from '@sap/cloud-sdk-vdm-business-partner-service';

@Injectable()
export class BusinessPartnerService {
  public dummyText = 'Not yet implemented.';

  getAllBusinessPartners(): any {
    return BusinessPartner.requestBuilder()
    .getAll();
  }
}
```

Requests can be executed with the `execute` function. To do this, the request builder needs to know where to send this request, in the form of a `Destination`. In this tutorial, we will provide this information directly to the `execute` function. We have described how to integrate with SAP Cloud Platform's Destination Service in [this tutorial](s4sdkjs-deploy-application-cloud-foundry). Every destination requires a URL.
Note, that we do not need the full URL to the service, but only the host of the system. Suppose, you have an SAP S/4HANA Cloud system running under `https://my.s4hana.ondemand.com/`, you can pass that information directly to the request builder like this (for simplicity we omit the imports and class definition from the previous code snippets):

```JavaScript / TypeScript
getAllBusinessPartners(): Promise<BusinessPartner[]> {
    return BusinessPartner.requestBuilder()
      .getAll()
      .execute({ url: 'https://my.s4hana.ondemand.com/' });
  }
}
```
Note that the response is properly typed as a promise containing a list of business-partners.

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 5: ](Add authentication to the request)]

Typically, you will need to authenticate yourself against a system in order to successfully execute a request. If you have a technical user for your system, you can pass the credentials like this:

```JavaScript / TypeScript
getAllBusinessPartners(): Promise<BusinessPartner[]> {
  return BusinessPartner.requestBuilder()
    .getAll()
    .execute({
      url: 'https://my.s4hana.ondemand.com/',
      username: "USERNAME",
      password: "PASSWORD"
    });
}
```

Alternatively, if you want to use the sandbox of the [SAP API Business Hub](https://api.sap.com), you will need to provide an `APIKey`. `withCustomHeaders` allows you to add custom HTTP headers to the request. You can pass your API key like this:

```JavaScript / TypeScript
getAllBusinessPartners(): Promise<BusinessPartner[]> {
  return BusinessPartner.requestBuilder()
    .getAll()
    .withCustomHeaders({
      APIKey: 'YOUR-API-KEY'
    })
    .execute({
      url: 'https://sandbox.api.sap.com/s4hanacloud/'
    });
}
```

In the examples above the `username`, `password` and `APIKey` are hard coded. This is only done to keep this tutorial brief. You should have a look at the [destination API](https://sap.github.io/cloud-sdk/docs/js/features/connectivity/destination) of the SAP cloud SDK to learn how to avoid hard coding credentials. Please do not use hard coded credentials in any productive solution.

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 6: ](Add select to the request)]

Like SQL, OData allows to only select specific properties of an entity. For our address manager, we only want to know the ID, the first name and the last name of a business partner. Add a select statement to your request like this:

```JavaScript / TypeScript
getAllBusinessPartners(): Promise<BusinessPartner[]> {
  return BusinessPartner.requestBuilder()
    .getAll()
    .select(
      BusinessPartner.BUSINESS_PARTNER,
      BusinessPartner.FIRST_NAME,
      BusinessPartner.LAST_NAME
    )
    .execute({
      url: 'https://my.s4hana.ondemand.com/'
    });
}
```

As you can see, each property we select is represented by an object on the `BusinessPartner` entity. If you type `BusinessPartner.` in your IDE, you will see all the properties the can be selected on that entity. This saves you from having to look up the properties in the metadata, and prevents errors due to mistyping.

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 7: ](Add a filter to the request)]

Business partners can either be natural persons or legal persons (e.g. organizations or companies). For the address manager, we only want the addresses of natural persons. Therefore, we need to a filter to our request. Modify your code like this:

```JavaScript / TypeScript
getAllBusinessPartners(): Promise<BusinessPartner[]> {
  return BusinessPartner.requestBuilder()
    .getAll()
    .select(
      BusinessPartner.BUSINESS_PARTNER,
      BusinessPartner.FIRST_NAME,
      BusinessPartner.LAST_NAME
    )
    .filter(
      BusinessPartner.BUSINESS_PARTNER_CATEGORY.equals('1')
    )
    .execute({
      url: 'https://my.s4hana.ondemand.com/'
    });
}
```

As for `select`, you can use the properties of the `BusinessPartner` entity directly for filtering. Each property offers a set of functions for constructing filters. Every property has the `equals` and `notEquals` function. Depending on that type of the property, there can be additional functions like `greaterThan` or `greaterOrEqual`. Also, since we know the type of the property, the VDM will prevent you from passing values of the wrong type. For example, `BusinessPartner.FIRST_NAME.equals(1)` would not compile (in pure JavaScript the code would only fail at runtime, but most editors will still raise a warning for the type mismatch).

[DONE]
[ACCORDION-END]

---
