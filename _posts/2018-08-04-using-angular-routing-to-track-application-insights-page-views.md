---
layout: post
title: Using Angular Routing to Track Application Insights Page Views
date: 2018-08-04
categories: [Azure, Application Insights]
---

# Using Angular Routing to Track Application Insights Page Views

## Introduction
Recently I created a Pluralsight course demonstrating how to implement Azure's Application Insights SDK within an Angular 6 UI and Node.js server applications. Doing just this one step provides some great telemetry information, but does miss some important telemetry information such as page views. Three possible options are briefly mentioned in the training. In order to not bog the training down too much with details of the Angular framework, the quick and "dirty" approach was demonstrated.  

Sometimes in software development one just needs to get something working. While the demonstrated approach will get the job done, it soon becomes tedious when there are many pages to be managed. 

In this post, I would like to dive a little deeper into one of the other options that was mentioned. Specifically in regards to using the Angular RouterModule. 

Note: I will not be digging into how to install the SDK into an Angular project. That is already being covered in the Pluralsight course, and in Microsoft's documentation. 

## Background
While doing the research for the Pluralsight course, I came across [this great article](https://toddmotto.com/dynamic-page-titles-angular-2-router-events) discussing an option for managing page titles using route change events. 

This post was written specifically for Angular 2, which uses an earlier version of the RxJS library that was based on returning promises. The newest library now returns an Observable which can be processed through "pipeable operators". The [RxJS GitHub documentation](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md) provides an explanation of how this works and why this is a better approach.

After switching over to using the new "pipeable operators" approach, all that had to be done was swap the setTitle() call for the AppInsights.trackPageView call. 

## Implementation
### Configuration
A working example of this approach can be found at https://github.com/HopperTech/ApplicationInsights-Demos/Angular-AppInsights-TrackPageViews

This project was created using the Angular CLI command ```ng new --routing``` The routing flag will add and configure the app-routing.module.ts to the base project. The events off of this routing module provide the key to making this approach work. 

In addition to the base project, two components were added to be able to navigate between. Simply called page-one and page-two. In order to be able to navigate between these, they were added to the routes configuration in app-routing.module.ts. 

```JavaScript
const routes: Routes = [
  {
    path: 'pageone',
    component: PageOneComponent,
    data: { title: 'Page One' }
  },
  {
    path: 'pagetwo',
    component: PageTwoComponent,
    data: { title: 'Page Two' }
  }
];
```

Notice the data property where a title is being passed. This property can contain a custom object that will be passed along with the selected route. Another thing I like about using this approach is that the page view names are managed declaratively in one location. There is no guessing or digging into code to see how a page will show up in Application Insights. 

While the demo replaces the setTitle call with the trackPageView, one could easily keep both commands. Using either the same title property, or even providing a separate property to be used for the AppInsights pagename. 

### Calling
Now to identify when to call the appInsights.trackPageView()

```JavaScript
export class AppComponent implements OnInit {
  constructor (
    private router: Router,
    private activatedRoute: ActivatedRoute
  ) {
    if (!AppInsights.config) {
      //Setup Application Insights within the Angular Application
      AppInsights.downloadAndSetup({
        instrumentationKey: environment.appInsights.instrumentationKey,
        enableCorsCorrelation: true
      });
    }
  }

  ngOnInit(): void {
    //Filter down to the correct event
    this.router.events
    .pipe(
      filter((event) => event instanceof NavigationEnd),
      map(() => this.activatedRoute),
      map((route) => {
        while (route.firstChild) {
          route = route.firstChild;
        }
        return route;
      }),
      filter((route) => route.outlet === 'primary'),
      mergeMap((route) => route.data)
    ).subscribe((event) => {
      AppInsights.trackPageView(event['title']);
    });
  }
}
```

Todd already provided a great explanation of how this filter works, so I encourage you to visit [his article](https://toddmotto.com/dynamic-page-titles-angular-2-router-events) for a detailed explanation. I just wanted to show you what the updated "pipeable" version would look like, and also the change to making the AppInsights.trackPageView() call. 


## Conclusion
I wanted to thank Todd for sharing his example of how to modify the page title in an Angular application. This is a great example of one of the things I really enjoy about software development, especially in this age of search engines. One can stand on the shoulders of work done before for both learning and modification for ones own intent. Though, I would encourage you to not just fall into the "copy, paste, and hope" mentality. But really dig into examples to understand them and make them your own.

If you are interested in watching the full discussion of implementing Application Insights within  Angular 6 and Node.js projects, I will post the link here once that has been published.