# POC Spartacus

## What

- Angular framework which auto-builds a client for SAP ECommerce backend
- Technology:
  - uses Angular's manual templating and component resolving under the hood
  - start-up is nearly zero-configuration when using schematics
  - after start-up the sample store can already be viewed which is impressive
  - dd

## POC

  - [get-started-video](https://www.youtube.com/watch?v=dEnIkBqEPuA)
    - start with Angular 15
    - `ng add @spartacus/schematics`
      - **FAIL**: `NOT SUPPORTED: keyword "id", use "$id" for schema ID`

## Analysis

 - technology
    - uses [own outlet structural directive](https://github.com/SAP/spartacus/blob/4bc9138a6ebf0a91cdd7bfc1ab9b26f9e226bc3c/projects/storefrontlib/cms-structure/outlet/outlet.directive.ts)
        - is *probably* overkill to use that instead of template refs and `ng-content`
            - but may be needed due to auto-resolving components-on-network-layout-load
        - high-maintanenace decision in terms of Angular updates (`Renderer` internals used there change relatively often)
    - uses `ngrx` for data flow which is by now debatable as it adds an awful lot of boilerplate to something which can be solved with Angular native DI tokens and reactive code
        - to be fair the business case at hand is a good one in favour of a state management library
    - uses `forRoot` providers a lot which is a good thing, but the configuration objects have to be manually cast in typescript which is not ideal
- approach
    - uses layout-per-network approach where the layout tree is loaded from the backend
        - good idea for standard looking apps
        - flexibility is accomplished by ([see video](https://www.youtube.com/watch?v=3xhnYUK9-sc)):
            - check a network layout request
            - find layout node that needs to be overridden
            - find the `flexId` to know which component it is in Spartacus
            - extend the Spartacus component in your component
            - inject in constructor services that you need e.g. `CurrentProductService`
            - write template with data from services
            - configure local module to override the Spartacus component with local component
        - the Spartacus components are open source which makes this simpler than it could have been

## Summary

Spartacus as an idea is interesting and zero-configuration start-up is impressive. 

### Technology

Some decisions are subjectively questionable but none seem wrong outright.

---
### Approach

Loading both layouts and data from backend and having standard components for all layout nodes works well in a rarely-updated business case and/or technological environment.

#### **Business Environment**

SAP ECommerce seems to be built as-a-standard so supporting older apis is very probably a long game while introducing new apis happens relatively rarely as business cases for digital storefronts are well known by now.

#### **Technological Environment**

Backend is built on SAP ECommerce which should be stable enough for this approach. Frontend however depends on evolving non-controllable runtime (browsers are updated at least once a month) and ever evolving security threats which need to be addressed as soon as possible by the frameworks involved. Meaning: using a library for a standard communication channel with the SAP backend is a good idea but using a 3rd party framework (Angular) for actual UI presentation in the browser is a horrible idea. The communication library won't change for years except for smaller bugfixes but the Angular (or any other) UI framework and it's runtime's evolutions are not under your control which means if there is no active maintenance then the project is doomed from the start.

---
### Learning Curve

As is often the case with SAP related products the Spartacus documentation exists but is not enough which means the developer in question should attend a practical SAP course because it incorporates an awful lot of implicit knowledge about the processes involved.

Examples:
- how does theming work?
- how to extend single components?
- how to extend DI sub-trees?
- how to extend state and side-effects?
- what are the (many) configuration levels and their configuration fields do?

---
### Framework State


## Verdict

Spartacus offers a quick-start into ECommerce storefront apps and by using Angular is supposed to also be compatible with PWA (offline available apps), SSR (server-side-rendering preload) and subsequent SEO optimizations while providing a comprehensive extension mechanic using Angular's runtime.

While Spartacus looks impressive when looking at a couple of videos and the technological approach it does not seem to have the required resources to support it. This makes it very dangerous to use as it seems to have many symptoms of a project which is soon to be closed.

- indifferent
    - PWA support: there is (assumption) no need for PWA at KION
    - SSR support: there is (assumption) no need for SSR at KION (reason: no user conversion)
    - SEO support: there is no need for SEO at KION (reason: no user conversion)
- problematic
    - the supported Angular version is 12 while the current version is 15
        - *symptom: not enough internal interest in product*
        - this alone means that **no new applications should use Spartacus** (Angular framework is not a library, using an older version means you are behind with all the security issues and browser modernization including all 3rd party libraries used there)
    - github issues: counting 702, oldest open is from 2018
        - *symptom: not enough internal support*
    - github prs: counting 347
        - *symptom: not enough internal support*
    - changelog: does not exist, instead there are releases which sadly don't describe the changes in a sufficient manner
        - *symptom: small closed off community relying on special knowledge instead of spreading the news*

---
### So, should I use Spartacus?

Right now the answer is **definitely not** because there is no (documented) easy way to create a Spartacus application with the newer versions of the main underlying framework.

#### **Assumption**: Spartacus gets a release with at least Angular 14.

Next threat: seemingly low interest in this project from the point of view of the internal team which leads to the presumption that the project won't survive and is only kept alive to make sure that existing customers don't have an immediate problem.

#### **Assumption**: Repo activity is greatly improved and issues and pull requests are resolved in time.

Any developer actively using Spartacus needs an official course, at the very least the developer who starts the project.

#### **Assumption**: Developer was introduced and is can begin in earnest.

OK, the project can be started. Assuming that the flexibility of extending the Spartacus storefront works even with very advanced business custom requests (that's a big assumption) this would be a viable approach to app development for SAP ECommerce.

#### **Assumption**: Spartacus can't be used anymore.

Switching from Spartacus to a custom implementation means a complete rewrite of the application. None of the code used to customize business cases when relying on Spartacus can be taken over except maybe for some presentation (i.e. "dumb") components. This means that when committing to Spartacus the developers need to have very high expectations regarding the repo activity, project longevity, stability and support-life of the project (right now this does not seem to be the case).

#### **Assumption**: new application without Spartacus.

There is no research involved in this POC regarging ECommerce backend itself so it's not possible to predict the usual challenges here e.g. authentication, api contract update processes, implementation intricacies etc. Given that SAP ECommerce is a standard product and there is consulting available this should not present any unsolvable or even particularly challenging problem. Additionally the product has less possible high-maintenance problems when comparing to open-internet storefronts as all the customers are already converted (no need to think about PWA, SSR, SEO), we are mostly talking about an intranet application looking like a storefront and maybe deployed on open internet but without the usual user-conversion hassle. All the usual self-made advantages apply i.e. any customization can be done manually without being restricted by 3rd party technical approach and experimenting is easy. In comparison to Spartacus the big disadvantage is the longer start-up time but given that there are a lot of storefronts which can provide insights into design and approach this is not a "don't know how" problem but a "let's work through it" challenge. A lesser disadvantage is the fact that Spartacus gets proposed layouts from the backend which means backend can change those layouts anytime according to current guidelines and the frontend code does not need to be adjusted in most cases. This is an interesting case where it should be researched whether it is possible to read the provided layout endpoints without using Spartacus and build the local custom layout with local components completely.