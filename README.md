Welcome to the IntelliJ IDEA UI test library project! Here you'll find several pieces of information and advices on how to set up, use and contribute to this library.

## Purpose of this project
This project allows you to create automated UI tests for your IntelliJ IDEA plugin project. Using this library you are able to access UI elements such as buttons, inputs, tree elements etc. and perform actions with them. Navigating through wizards, clicking on buttons or editing file content of newly created project could be automated using this library.

## Any Suggestions or Questions?
Please submit an [issue](https://github.com/redhat-developer/intellij-common-ui-test-library/issues) to this project.

## Contributing
Feel free to contribute to this project! See the [contribution guide](https://github.com/redhat-developer/intellij-common-ui-test-library/blob/main/CONTRIBUTING.md) for more details.

## Quick setup
The setup of this library is easy - just extend the **build.gradle.kts** file as described in the following steps, and you are ready to write your first UI test.

### STEP #1: Adding repositories
You need to add the following nexus and JetBrains repositories:
```
repositories {
    maven {
        url 'https://raw.githubusercontent.com/redhat-developer/intellij-common-ui-test-library/repository/'
    }
    maven {
        url 'https://packages.jetbrains.team/maven/p/ij/intellij-dependencies'
    }
}
```

### STEP #2: Adding dependencies
Add the following dependency:
```
dependencies {
    compile 'com.redhat.devtools.intellij:intellij-common-ui-test-library:0.4.4'
}
```

### STEP #3: Adding source sets
The following source set is needed to define where in your project will be your UI tests and resources located. The following example displays the 'src/it/java' location for java code of UI tests and the 'src/it/resources' location for resources:
```
sourceSets {
    integrationTest {
        java.srcDir file('src/it/java')
        resources.srcDir file('src/it/resources')
        compileClasspath += sourceSets.main.get().compileClasspath + sourceSets.test.get().compileClasspath
        runtimeClasspath += output + compileClasspath + sourceSets.test.get().runtimeClasspath
    }
}
```

### STEP #4: Adding tasks
```
task integrationTest(type: Test) {
    useJUnitPlatform()
    description = 'Runs the integration tests.'
    group = 'verification'
    testClassesDirs = sourceSets.integrationTest.output.classesDirs
    classpath = sourceSets.integrationTest.runtimeClasspath
    outputs.upToDateWhen { false }
    mustRunAfter test
}

runIdeForUiTests {
    systemProperty "robot-server.port", System.getProperty("robot-server.port")
}
```
## Additional Features

### Creating a Project

**Creating an Empty Project:**
Use the following method to create an empty project with a specified name:

```java
CreateCloseUtils.createEmptyProject(remoteRobot, "empty-test-project");
```
**Creating a New Project with a Specific Type:**
You can also create a new project with a specific type, such as Java, Maven, or Gradle:
```java
CreateCloseUtils.createNewProject(remoteRobot, "new-test-project", CreateCloseUtils.NewProjectType.PLAIN_JAVA);
```
### Test project location
Default test project location is `/home/user/IdeaProjects/intellij-ui-test-projects/`.
Developers can specify the location where the test project will be created by providing a system property called `testProjectLocation`. For example:
```
task integrationTest(type: Test) {
    ...
    systemProperties['testProjectLocation'] = '/home/user/IdeaProjects/intellij-ui-test-projects/'
    ...
}
```
Or add the location as a parameter for gradlew command which runs the test. For example:
```
systemProperties['testProjectLocation'] = project.hasProperty('testProjectLocation') ? project.property('testProjectLocation') : null
    
./gradlew integrationTest -PtestProjectLocation=${env.HOME}/IdeaProjects/intellij-ui-test-projects/
```

### Remote-robot IntelliJ instance logs
If developers want to view the remote-robot intellij instance logs, they can specify system property intellij_debug to save these logs to files, which will be stored inside $user.dir/intellij_debug folder.
```
task integrationTest(type: Test) {
    ...
    systemProperties['intellij_debug'] = 'true'
    ...
}
```

## Start and quit IntelliJ IDEA
Use the following code to start IntelliJ before running the first UI test. The runIde() method not only starts the IDE for UI tests, it also returns reference to the Remote-Robot instance which will be useful later to access UI elements such as buttons, inputs etc.
```
private static RemoteRobot robot;

@BeforeAll
public static void runIdeForUiTests() {
    robot = UITestRunner.runIde(UITestRunner.IdeaVersion.COMMUNITY_V_2022_2, 8580);
}
```

After executing all the UI tests close the IDE by running the following command:
```
@AfterAll
public static void closeIde() {
    UITestRunner.closeIde();
}
```

## What next? Implement your first UI test!
After you manage to set up this library to your project and successfully start and quit IntelliJ IDEA, there is no more setup needed. Just start writing your UI tests! Here are some examples that will help you get started:

### Create your first fixture
Create an instance of a FlatWelcomeFrame class which allows you to access the 'Welcome to IntelliJ IDEA' dialog's UI.
```
FlatWelcomeFrame flatWelcomeFrame = remoteRobot.find(FlatWelcomeFrame.class, Duration.ofSeconds(10));
```

### Use the fixture to access UI
After you have the object, it can be used to access the UI - here is an example of clicking on the 'New Project' button:
```
flatWelcomeFrame.createNewProject();
```

### Use utilities
This library provides several static utility methods for common actions. Here is an example of one useful transformation method:
```
String contentListStr = TextUtils.listOfRemoteTextToString(contentList);
```

### Use any tool provided by Remote-Robot framework
Besides the fixtures and utilities provided by this library you can use any tool from the [Remote-Robot](https://github.com/JetBrains/intellij-ui-test-robot) framework itself. 

