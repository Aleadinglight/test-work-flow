
# Using Github Action To Automate Workflow

Personal notes on how to set up github action as a CI/CD for automating workflow.

## Setting up github page
Go to **Repository settings > Pages**, choose a branch and folder for hosting. Mine is on branch master and /docs folder.

Create another branch, for example here **/dev** branch, and checkout this branch.
This will the main workspace in our local environment. Any new features or hot-fix will be committed to this **/dev**, and we will set up the **Github Action** to automate the deployment to **/master** branch. 

## Set up local development
Create react application with `create-react-app`:
```bash
npx create-react-app my-app
cd my-app
npm start
```

Add a `homepage` attribute to your `package.json` for relative path.

```json
{
	"name": "my-app",
	"version": "0.1.0",
	"homepage": ".",
	"private": true,
	...
}
```

## Setting instructions for our workflow 
Clone project and checkout **/dev** branch. 

Create a new github file in: `.github/workflows/test-push-action.yaml`. Here under `.workflows/` folder we can have multiple action files for multiple different processes.

Some key notes:

-	The workflow instruction for a trigger in branch **/dev** must be place inside branch **/dev**. Same for other branches.
-	Each action can contain multiple **jobs**. These job run in different workspace, so you have to use **actions/checkout@v2** for each of them. The environment will also different so you need to also run **actions/setup-node@v2** and other environment settings (if exist) again. 
-	Each **job** can contains multiple **steps**. These step is execute one-by-one and you can add condition to handle failure of the previous step.

Some workflow action requires token for authorization. These tokens can be added via **Repository settings > Secrets**. 

Because we don't want interactive mode when we run our test via Github Action, we would set our test command to `npm test -- --watchAll=false` in our `.yaml` file.

We also need to change the `build/` folder name to `docs/` for hosting purpose. The easiest way to do it is to rename it in the **terminal**. So something like this will do.
```yaml
	- name: Build and test
		run : |
			npm install
			npm run build
			npm test -- --watchAll=false
	- name: Set up Github Page source folder
		run : |
			rm -rf docs/
			mv ./build ./docs
```

## The process
- Every time someone push to **/dev** branch -> The workflow is triggered.
- The repository is checkout in a specific environment.
- Build & test the new updates.
- Some third party tools do some check up and optimization.
- If everything is fine, the changes is pushed to the master branch and deployed to the project page.
