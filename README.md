# <span style="color:#ADD8E6"> Lab5 - CD/CI with GitActions </span>

<div align="right"> </div>

## <span style="color:#ADD8E6">Table of Contents </span>

- [Description](#desc)
- [Prerequisites](#pre)
- [Background](#bg)

<a id="desc"></a>

## <span style="color:#ADD8E6"> Description </span>

This is the start of the last series of labs you will see for CSCC01. You have gone through the motions of building a full-stack application, including its database, backend and frontend. Your last job in the process of getting an application up and running will be the process of _CI/CD_, or _continuous integration and continuous delivery:_

- **_Continous integration_** consists of building upon existing code frequently and reliably, usually with automated tests that ensure code changes are reliable. When working alongside multiple people, CI is an important part of the software development process that keeps code clean.
- **_Continous delivery_** automates the delivery of those changes to production environments, ensuring rapid and reliable software releases.

Lab 5 will consist of _continous integration_ (CI), where we will use _GitHub Actions_ (GitHub's native CI/CD platform) to implement _workflows_ that will run tests on our code. For this lab, we will design:

- The workflow file necessary for GitHub to build a virtual machine that will use our code, start our backend and run tests (for more detail, visit [this page](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration#about-continuous-integration-using-github-actions)).
- Some tests that will ensure the integrity and validity of our backend endpoints.

Lab 6 will handle the _continuous delivery_ (CD) portion, where you will be responsible for automating the build of a Docker image containing your code; a container that will be pushed automatically to a hosting platform and made public for the world to see.

![](/images/1.png)

<a id="pre"></a>

## <span style="color:#ADD8E6"> Prerequisites </span>

The prerequisited remain the same as the previous labs: we will make use of Node.js, MongoDB, a GitHub account and an IDE that will facilitate seeing our changes.

<a id="bg"></a>

## Implementing CI

A lot of the visualization for this lab will be shown through GitHub Desktop, but it is not necessary for this lab.

1. Create a new GitHub repository, with `C01Lab5` as its name, and ensure it is public. **_Do not fork this repository,_** since we will be setting up _rulesets_ for our repository from scratch (seen later in the lab). Clone the repo locally.
   ![](/images/2.png)
2. Copy the `quirknotes` folder and the `.gitignore` files _from **this** repository_ onto **_your_** newly-created repository. Make a commit with these changes to a `main` branch, and push it to the remote repository.
   ![](/images/3.png)
3. With our new changes, travel to the `quirknotes/backend` folder. Throughout the lab, we will use [Jest](https://jestjs.io/) as the framework that will facilitate testing our backend.
   Install `Jest` using:
   ```
   npm install jest --save-dev
   ```
   Once finished, go inside the `package.json` file, and replace the `test` script with running Jest:
   ```javascript
   ...
   "scripts": {
    "test": "jest",
    ...
   },
   ```
4. Inside of our `backend` folder, create a `tests` folder, and a file named `status200.test.js`. Jest automatically targets and runs any file with a `*.test.js` extension. We can test as such by populating the file with the following code:

   ```javascript
   test("1+2=3, empty array is empty", () => {
     expect(1 + 2).toBe(3);
     expect([].length).toBe(0);
   });
   ```

   - The `test` function always contains a title for the test, a function to run and some [matcher(s)](https://jestjs.io/docs/using-matchers) to test values (in this case, a series of `expect` matchers).

   You can run the tests by running `npm test` on the terminal:

   ```
   > backend-example-2@1.0.0 test
   > jest

   PASS  tests/status200.test.js
   √ 1+2=3, empty array is empty (5 ms)

   Test Suites: 1 passed, 1 total
   Tests:       1 passed, 1 total
   Snapshots:   0 total
   Time:        1.441 s
   Ran all test suites.
   ```

5. From here, we will make actual use of our backend to run a test on an endpoint. On another terminal, get a local MongoDB instance and our `server.js` file running (as shown in [previous labs](https://github.com/ArmandoRJr/c01w24lab3?tab=readme-ov-file#db)). Please ensure that the database is clean (i.e. there are no notes)!
   Afterwards, we can append the following code to our file:

   ```javascript
   const SERVER_URL = "http://localhost:4000";

   test("/getAllNotes - Return list of zero for getAllNotes", async () => {
     const notesRes = await fetch(`${SERVER_URL}/getAllNotes`, {
       method: "GET",
       headers: {
         "Content-Type": "application/json",
       },
     });

     const notesBody = await notesRes.json();
     const notes = notesBody.response;

     expect(notesRes.status).toBe(200);
     expect(notes.length).toBe(0);
   });
   ```

   This is similar to how our frontend would handle a response from our backend, using `fetch` to ping the backend, but we will instead use the response to verify that a) the API returns the right response and b) the `notes` array is of the right length given no notes are in the database.
   Run `npm test` again, and verify the code works:

   ````
    PASS  tests/status200.test.js
    √ 1+2=3, empty array is empty (4 ms)
    √ /getAllNotes - Return list of zero for getAllNotes (113 ms)

    Test Suites: 1 passed, 1 total
    Tests:       2 passed, 2 total
    Snapshots:   0 total
    Time:        0.775 s, estimated 1 s
    Ran all test suites.
    ```
    (If the 2nd test case does not pass, verify your backend is running and that MongoDB is working correctly.)
   ````

   Commit your code so far and push it to the repo.
   ![](/images/4.png)

6. Go back to GitHub, and click on the `Actions` tab. We will set up our first workflow for automating code testing using Node.js and the Jest test suite we have developed so far. Search for `Node.js` on the search bar, and click `Configure` on the workflow provided by GitHub Actions.
   ![](/images/5.png)

   This leads to a Node.js workflow template.
   Before breaking down each component of this template, it is **_heavily encouraged_** to read the [About workflows](https://docs.github.com/en/actions/using-workflows/about-workflows) page from GitHub Actions to understand workflow basics (up until the end of `Understanding the workflow file`).
   The breakdown proceeds as follows:
   **_[Add breakdown here]_**
   We can press `Commit changes...` on the top right to add the YAML file to our repository (commiting directly to the `main` branch).
   If we go to the `Actions` tab again, we can see that our workflow was ran once, with three different jobs that have _failed._ This is normal, as our workflow has not been configured correctly. We will fix this soon, but you are free to explore the steps that were ran during the process.
   ![](/images/6.png)

7. Go to the `Settings` tab, then under `Rules` click `Ruleset`. [Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) help you to control how people can interact with branches and tags in a repository. We will set up rules that prevent direct commits to our main branch, such that pull requests that pass certain workflow jobs will be required to make any changes.
   Configure a ruleset with the following settings:

- Ruleset Name: `Change main only through PRs`
- Enforcement status: `Active`
- Target branches: `Include default branch`
- Branch protections: `Restrict deletions`
- Require a pull request before merging
  - Required approvals: 0
- Require status checks to pass
  - Add checks: `Backend test (20.x)`
- Block force pushes
  Save your changes.

8. `git fetch` and `git pull` the changes made to the remote repository into your local repository. From the `main` branch, make and checkout into a new branch named `fix/workflow-changes`. Replace the code inside the `.github/workflows/node.js.yml` file with the following starter code:

   ```yaml
   name: Node.js CI

   on:
   push:
       branches: ["main"]
   pull_request:
       branches: ["main", "dev"]

   jobs:
   build:
       name: Backend test
       runs-on: ubuntu-latest

       defaults:
       run:
           working-directory: ./quirknotes/backend

       strategy:
       matrix:
           node-version: [20.x]

       steps:
       - uses: actions/checkout@v3

       - name: Use Node.js ${{ matrix.node-version }}
           uses: actions/setup-node@v4
           with:
           node-version: ${{ matrix.node-version }}

       - run: npm install
       - run: npm test
   ```

   **_[Add small breakdown here]_**
   Commit the changes, and publish the branch.
   Given this new branch, go back to GitHub and open up a pull request.
   You may catch a glimpse of the above workflow being prepared.
   ![](image.png)
   The name of the job matches the status check outlined in our ruleset, a job that _must be successful_ if the PR is to be merged to `main`.
   However, this job is set to fail: our basic arithmetic + array check test passes, but our backend test will crash given _our backend is not running in the GitHub Actions virtual machine._ Your first task for this lab is to fix that.
   ![](/images/8.png)

---

### Task #1: Fix GitHub Actions workflow

The YAML file outlining how the workflow jobs should be set up are missing steps for:

- Installing and running a local MongoDB instance
- Running our `server.js` file while our tests are being ran

Change the workflow's YAML file to fix that.

- You may make use of other GitHub Actions to aid your workflow.
- You may also change the name of the status check in our ruleset, to accomodate for the changes above.

You will only need to edit the YAML file. Do not modify the tests themselves.
When your changes are successful, a commit with these changes will trigger a (now successful) re-run of the workflow, and will let you merge the PR with main.
![](/images/9.png)

Finish the task by merging the pull request, and deleting the `fix/workflow-changes` branch.

---

9.  With our workflow fully set up, we can set up a `dev` branch stemming from `main` for any future changes that may be made, following the Gitflow structure (as seen last lab).
    From this `dev` branch, make a branch with the name `feat/test-coverage`.
    Once in this branch, add the following code to your `status200.test.js` file:

    ```javascript
    test("/getAllNotes - Return list of two notes for getAllNotes", async () => {
      // Code here
      expect(false).toBe(true);
    });

    test("/deleteNote - Delete a note", async () => {
      // Code here
      expect(false).toBe(true);
    });

    test("/patchNote - Patch with content and title", async () => {
      // Code here
      expect(false).toBe(true);
    });

    test("/patchNote - Patch with just title", async () => {
      // Code here
      expect(false).toBe(true);
    });

    test("/patchNote - Patch with just content", async () => {
      // Code here
      expect(false).toBe(true);
    });

    test("/deleteAllNotes - Delete one note.", async () => {
      // Code here
      expect(false).toBe(true);
    });

    test("/deleteAllNotes - Delete three notes.", async () => {
      // Code here
      expect(false).toBe(true);
    });

    test("/updateNoteColor - Update color of a note to red (#FF0000)", async () => {
      // Code here
      expect(false).toBe(true);
    });
    ```

    Save the file and commit the changes to the branch. Push the changes to remote, and make a pull request from `feat/test-coverage` to `dev`.
    You will notice that, although checks are not _required_ for commits to the `dev` branch, they will still run when making any pull request. In our case, most of our tests will fail and the check will fail as a result. The task is now to fix the tests.

---

### Task #2: Fix test coverage

The tests outlined have yet to be completed for proper coverage.
You are responsible for completing the tests such that they check for an appropriate response from all endpoints. At the minimum, they should all check for a `200` status when successful. Use `npm test` for running your tests locally.

- You may have more than one `expect` matcher (or other types of matchers) to satisfy the condition(s) required by the test.
- Remember each test is supposed to be independent. There should be no need to have information from one test affect another.
- Most of these test cases allow for checking both the status and the right body response (e.g. `[number of notes] notes deleted.`). It is encouraged you cover this in your test cases too.
- You may find that one of these tests actually highlights erroneous code present in our backend. You may fix it after finding it within this PR.

---

After completing the above, commit and push your changes. The automated tests should now pass, and the `Merge pull request` button should be green instead of gray. Do not actually merge the PR at the end.
