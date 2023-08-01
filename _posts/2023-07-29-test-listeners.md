---
layout: post
title: Using test listeners with java and junit5 for component tests
---
I have been refactoring one of the container sidecars owned by my team, focusing on the component tests.

 These tests were deploying the database, a kafka server and the application itself. One of the objectives in this refactor was to keep the start of the database and kafka at the beginning of any test execution and close it at the end, while using Junit5. We could do it because our application just read from the database, so tests will not modify the state.
In java, you can do some stuff before and after the tests defined in one class with Junit annotations @BeforeAll and @AfterAll, but here we could not do it because, for the sake of maintainability, tests were grouped in different classes based on types of tests. Thus, after some research, we develop two tests listeners that were executed before and after the selected  tests were executed (a single test, a class, a group of classes or all...)

## The dependency
Tests listeners are part of junit, we included the dependency below in our pom:

```
<dependency>
  <groupId>org.junit.platform</groupId>
  <artifactId>junit-platform-suite-engine</artifactId>
  <scope>test</scope>
</dependency>
```

## How to use it
We created a class that was implementing the interface [TestExecutionListener](https://junit.org/junit5/docs/5.0.3/api/org/junit/platform/launcher/TestExecutionListener.html), as is shown below:

```
public class EnvironmentListener implements TestExecutionListener {
  private static final Logger logger = LoggerFactory.getLogger(EnvironmentListener.class);
  private static final CtDependencies ctDependencies = new CtDependencies();

  @Override
  public void testPlanExecutionStarted(final TestPlan testPlan) {
    logger.debug("Listener triggered: executing testPlanExecutionStarted");
    ctDependencies.startAll();
  }

  @Override
  public void testPlanExecutionFinished(final TestPlan testPlan) {
    logger.debug("Listener triggered: executing testPlanExecutionFinished");
    ctDependencies.stopAll();
  }
}
```

These two methods were executed at the beggining before any test was started (testPlanExecutionStarted)  and after all had finished (testPlanExecutionFinished). So basically we were starting all components and then stopping all, with the logic encapsulated in CtDependencies.

To enable the listeners, one more thing was needed, create the following file src/test/resources/META-INF/services/org.junit.platform.launcher.TestExecutionListener that had the route to our class implementing the methods:
ct.environment.EnvironmentListener

## Benefits
The benefit after all this was that we were spending a few seconds to set up the test environment just once instead of start and stop everything for a single test or for a class, having twenty test classes.
