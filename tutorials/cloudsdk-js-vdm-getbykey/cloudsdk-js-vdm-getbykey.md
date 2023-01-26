---
title: AAA for test Use OData Navigation Properties with the SAP Cloud SDK's Virtual Data Model QA Green changed 
description: Use OData navigation properties with the SAP Cloud SDK's virtual data model to duild an address manager application.
auto_validation: true
time: 20
tags: [ tutorial>intermediate, topic>javascript, software-product>sap-business-technology-platform, tutorial>community]
primary_tag: software-product>sap-trade-management
author_name: Test Test
author_profile: Oleksandra_Kovtunenko@epam.com
---

## Prerequisites
 - Have `Node.js` and `npm` [installed on your machine](s4sdkjs-prerequisites)
 - Access to an SAP S/4HANA Cloud system or the [SAP API Business Hub Sandbox](https://api.sap.com/getting-started), or use the [Business Partner Mock Service](https://sap.github.io/cloud-s4-sdk-book/pages/mock-odata.html)
 - Basic knowledge of OData is recommended, but not required

## Details

> ### We migrate tutorials to our [documentation](https://sap.github.io/cloud-sdk/)
> This tutorial is not actively maintained and might be partially outdated.
> Always up-to-date documentation is published on our [documentation portal](https://sap.github.io/cloud-sdk/).
> We will provide a link to the updated version of this tutorial as soon as we release it.
> In this tutorial, version 1 of the SAP Cloud SDK for TypeScript/JavaScript is used.

### You will learn
  - How to use the Virtual Data Model to get a single entity by its key
  - How to use the Virtual Data Model with OData navigation properties
  - How to expose this data in your `NestJS` application

The goal of this tutorial group is to show you how to implement a JavaScript application that allows you to manage the addresses of business partners. This application will be using `NestJS` and the SAP Cloud SDK for JavaScript. In this tutorial, we use the SAP Cloud SDK's OData Virtual Data Model to get a single business partner entity and all related addresses.

---

[ACCORDION-BEGIN [Step 1: ](Add an API endpoint)]

In a [previous tutorial](cloudsdk-js-vdm-getall) we explained a bit the basics about `controller`, `service` and `module` of `NestJS` applications. Note: If you have already controller and service classes from the previous tutorial you can of course keep the existing files and just extend the classes by the new methods. Since we want to expose a new endpoint, start by creating  a controller file called `business-partner.controller.ts` in the `src` folder of your project. Then, copy the following code into it:

```JavaScript / TypeScript
import { Controller, Get, Param } from '@nestjs/common';

@Controller('business-partners')
export class BusinessPartnerController {

  @Get('/:businessPartnerId')
  getBusinessPartnerByID(@Param('businessPartnerId') businessPartnerId): string {
    return `My id for query is: ${businessPartnerId}`;
  }
}
```

Then prepare a service file called `business-partner.service.ts` and add this dummy implementation:

```JavaScript / TypeScript
import { Injectable } from '@nestjs/common';
import { BusinessPartner } from '@sap/cloud-sdk-vdm-business-partner-service';

@Injectable()
export class BusinessPartnerService {
  getBusinessPartnerById(businessPartnerId: string): Promise<BusinessPartner> {
    return;
  }
}
```

Finally, register the `controller` and `service` in the root application module `app.module.ts`:

```JavaScript / TypeScript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { BusinessPartnerController } from './business-partner.controller';
import { BusinessPartnerService } ### from './business-partner.service';

@Module({
  imports: [],
  controllers: [AppController, BusinessPartnerController],
  providers: [AppService, BusinessPartnerService]
})
export class AppModule {}
```

Now you can restart your app with:
```Shell
npm start
```
And open `http://localhost:3000/business-partners/TestID`. You should see the response: `My id for query is: TestID`.

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 2: ](Get a single business partner)]

Next, you will implement the `getBusinessPartnerById` function to get a single business partner by its ID. Open `business-partner.service.ts` and add the following function to your code:

```JavaScript / TypeScript
import { Injectable } from '@nestjs/common';
import { BusinessPartner } from '@sap/cloud-sdk-vdm-business-partner-service';

@Injectable()
export class BusinessPartnerService {
  getBusinessPartnerById(businessPartnerId: string): Promise<BusinessPartner> {
    return BusinessPartner.requestBuilder()
    .getByKey(businessPartnerId)
    .execute({
      url: 'https://my.s4hana.ondemand.com/'
    });
  }
}
```

As before, we call `BusinessPartner.requestBuilder()` to select the type of request we want to build. Since we only want to get a single business partner, we use the `getByKey` function and then call `execute` to execute the request. Add credentials as needed as shown in the [previous tutorial](cloudsdk-js-vdm-getall).

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Add select to the request)]

As in the `getAll` case, we can again select the specific properties of the entity that we're interested in. Add a select statement to your code as shown below:
(for simplicity we omit the import and class declaration and focus on the method implementation)

```JavaScript / TypeScript
getBusinessPartnerById(businessPartnerId: string): Promise<BusinessPartner> {
  return BusinessPartner.requestBuilder()
    .getByKey(businessPartnerId)
    .select(
      BusinessPartner.BUSINESS_PARTNER,
      BusinessPartner.LAST_NAME,
      BusinessPartner.FIRST_NAME,
      BusinessPartner.IS_MALE,
      BusinessPartner.IS_FEMALE,
      BusinessPartner.CREATION_DATE
    )
    .execute({
      url: 'https://my.s4hana.ondemand.com/'
    });
}
```

[DONE]
[ACCORDION-END]


---
