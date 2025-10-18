# Clean Architecture User Login System Example

The following summarizes the structure of the code and highlights the structure of a use case.
This code will be used as an example in lectures and as the starter code for your next lab activity,
so please take the time to get familiar with the code this week.

## Getting a copy of the Code
**To get started exploring the code, fork this repo on GitHub and then make a clone.**

Open the project in IntelliJ and make sure you can successfully run `app/Main.java`.

> Note: you may need to set the Project SDK in the `Project Structure...` menu, and possibly also manually link the Maven project.

## Understanding the Program

Try the signup, login, and change password use cases by running the program.
Notice that the "Log Out" button doesn't do anything when you click it — to test whether
change password worked, you'll need to quit and rerun the program.

> Note: some other buttons, like the "Cancel" buttons, are also not fully functional.

### Packaging

Explore the package structure in `src\main\java\`. There are packages for
the CA layers — `view`, `interface_adapter`, `use_case`,
`entity`, and `data_access` — as well as `app`, a package for the main program
and code for building everything.

Two of these packages, `use_case` and `interface_adapter`, have subpackages for each of the
two use cases: `login` and `signup`. None of the Interactor and Interface Adapter code is
shared between use cases.

Several packages _don't_ have subpackages: `data_access`, `entity`, `view`, and
`app`.

* The same View object may have several Use Case buttons inside a single `JPanel`,
  so separating by use case isn't possible.
* Entities represent the data from the problem domain that all Use Cases manipulate.
* The Data Access layer is responsible for saving and reading the Entities.
* The main application is responsible for building the CA engine and starting the GUI.
  After the engine is built and the UI becomes visible, the program is driven by the user and
  the main program has nothing left to do.

### A note on English: verb phrases vs. nouns

"Sign up" is a verb phrase and "signup" is a noun. That generalizes: "check in"
vs "checkin", "log in" vs "login". Two words for the verb phrase, 1 word for the
noun phrase.

For example, to complete a login, you need to log in. (Say it out loud. They sound different.)

In "the login process", "login" is a noun acting as an adjective to describe
"process". "Basketball coach" is another example of this English construct.

### Comparing the signup and login code

Let's compare these two use cases.

#### Controllers

In IntelliJ, find `LoginController` and double click it to open it.

Now right-click on `SignupController` and select `Open in Right
Split`. When you do, you will see the two controllers side by side.
They are identical in structure, differing only in the
details.

**This is powerful:** most controllers will look similar. Most presenters
will look similar. Most interactors will look similar. Any programmer who
learns about the CA will have a good understanding of
any controller, interactor, and use case.

**Thought question**: open the CA Engine diagram and compare the types
in `LoginController` to the diagram. You'll notice that both controllers
have an Input Boundary that is _injected_ in the constructor, both create
Input Data from the parameters in method `execute`, and both
of them call the Use Case execute method, passing in the Input Data. All
the arguments for the `execute` method come from the View.

#### Presenters

Open `LoginPresenter` and `SignupPresenter` side by side. Both have
View Model variables and a View Manager Model that are injected into the
constructor.

Both also have a `prepareSuccessView` method that the Use Case calls
when it is complete. The job of this method is to update the View Models.
Read the code for this method in either presenter.

Notice that both of the `prepareSuccessView` methods mutate the state of a View Model
and call `firePropertyChanged` to alert the relevant View Model that
the state has changed, and ends with code that tells the View Manager Model
what the active View should be.

Both Presenters also have a `prepareFailView` method to handle errors.

#### Interactors

Now compare `LoginInteractor` and `SignupInteractor` side by side. (You can drag
tabs around if you like.)

**Thought question:** Why doesn't the `LoginInteractor` have a `UserFactory`
but `SignupInteractor` does?

A Controller calls the `execute` method in an Interactor to start processing
the Use Case data. When it's done, the Interactor tells its Presenter what the result
is, and the Presenter puts it into the View Model and tells the View Model Manager to change
which View is showing.

Compare `LoginInteractor` and `SignupInteractor`. Notice that both
use an Input Boundary, Input Data, Output Boundary, and
Output Data. Both also have a Data Access Interface, which is what the Interactor
uses to get data relevant to the Use Case.

The Data Access Interface and Output Boundary are injected in the constructor.

Method `execute` is passed Input Data to process. The Interactor fetches the
appropriate piece of persistent data from the Data Access Interface, does some error checking
to make sure the Use Case makes sense, and then does whatever the Use Case
is supposed to do. Notice the Interactors both end by creating Output Data and
telling the Presenter to present it.

## Data Access Object (DAO)

There are three DAOs in package `data_access`! All three implement the Data Access Interface
from the use cases. The Use Case code works with any of them.

* Class `FileUserDataAccessObject` manages data storage and retrieval in a
CSV file, and also keeps the data in a `Map` for easier access. This temporary storage
is called a *cache* of the information in the file.

* Class `DBUserDataAccessObject` uses okhttp to use an API, working with JSON data. Your team
  might want to refer to this when you do your API work. This API is similar to the one from lab;
  you can read its documentation
  [here](https://www.postman.com/cloudy-astronaut-813156/csc207-grade-apis-demo/documentation/fg3zkjm/5-password-protected-user). This is the DAO used initially in the `app.AppBuilder` class.

* Class `InMemoryDataAccessObject` doesn't save the user data to any kind of file at all,
  and is intended to be used by unit tests.
  * It's also simple to write, which means that you can start
    programming your Use Cases _before_ you even have the details of data persistence worked out!

> You can change which DAO is being used by modifying the `app.AppBuilder` code.

## Getting Familiar with the Code
The following two exercises are additional ways to confirm your understanding of the design of the code.

### Exercise 1: Tracing Execution of a Use Case interaction
- Set a breakpoint inside one of the action listeners for a button, like the "sign up" or "log in" buttons.
- Run the program in debug mode and perform the interaction.
- Step through the code to trace the flow of information through the program.
- Ensure you follow every step of the execution; it may be helpful to refer to a copy of the CA Engine diagram as you do this.

### Exercise 2: Tracing Creation of the CA Engine
- Set a breakpoint at the very first line of the body of the `Main.main` method.
- Run the program in debug mode.
- Step through the code to trace how the `AppBuilder` is used to build the whole program,
  with the various Views and associated Use Case Interactors getting pieced together.
- Ensure you follow every step of the execution; it may be helpful to refer to a copy of the CA Engine diagram as you do this.
