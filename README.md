# Blazor WASM Deploy Only
Deployed at: <a href="https://mariekauth.github.io/BlazorWasmDeployOnly">BlazorWasmDeployOnly</a>

## Instructions
1. Create a new empty repo on GitHub
2. Create a new directory for your repo (should be the name of your project)
3. Navigate to that folder and open your project in VSCode
4. In VSCode, open a "bash" terminal and run the commands there.

## Deploy a simple static site and deploy it
Replace GITHUB_ACCOUNT_NAME with the name of your github account.
```bash
GITHUB_ACCOUNT_NAME="mariekauth"
PROJECT_NAME=$(basename $PWD)
echo $GITHUB_ACCOUNT_NAME
echo $PROJECT_NAME
```
Ensure the PROJECT_NAME and ACCOUNT_Name are correct before continuing.


```
git init
echo "<html><head><title>Test ${PROJECT_NAME}</title></head><body><h1>Deployed ${PROJECT_NAME}</h1></body></html>" >> index.html
echo "# $PROJECT_NAME" >> README.md
echo "Deployed at: <a href=\"https://${GITHUB_ACCOUNT_NAME}.github.io/${PROJECT_NAME}\">$PROJECT_NAME</a>" >> README.md
git add .
git commit -m "Initial Commit"
git remote add origin "git@github.com:${GITHUB_ACCOUNT_NAME}/${PROJECT_NAME}.git"
git branch -M main
git push -u origin main
git checkout -b gh-pages
git push origin gh-pages
git checkout main

```
This might take a minute.

Your site should now be deployed to github pages.

The URL should be in your README.md file.

## Setup Your static Blazor App
The next script will:

- Remove the index page
- Add all the normal content, like .gitignore and License
- Add the Blazor App
- Deploy the Workflow

Run the script
```bash
rm index.html
dotnet new gitignore
echo "# $PROJECT_NAME" >> README.md
echo ".vscode" >> .gitignore
cp ../LICENSE LICENSE
dotnet new sln -n ${PROJECT_NAME}
dotnet new blazorwasm -n "${PROJECT_NAME}" -o "${PROJECT_NAME}" -f net6.0
dotnet sln "${PROJECT_NAME}.sln" add "${PROJECT_NAME}/${PROJECT_NAME}.csproj"
mkdir -p .github/workflows && touch .github/workflows/main.yml
code . .github/workflows/main.yml
```

## Update .github/workflows/main.yml and Deploy
Copy the following into .github/workflows/main.yml

Update Line 4: Replace PROJECT_NAME: BlazorWasmDeployOnly with the name of your project.
```yaml
name: Deploy To GitHub Pages
env: 
  # Set the Project Name Here
  PROJECT_NAME: BlazorWasmDeployOnly
on:
  push:
    branches:
    - main
permissions:
  contents: write

jobs:
  deploy-to-github-pages:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 6.0.x

    - name: Install .NET WASM Build Tools
      run: dotnet workload install wasm-tools

    - name: Publish .NET Core Project
      run: dotnet publish ${PROJECT_NAME}/${PROJECT_NAME}.csproj -c:Release -p:GHPages=true -o dist/Web --nologo

    # changes the base-tag in index.html from '/' to 'ProjectName' to match GitHub Pages repository subdirectory
    - name: Change base-tag in index.html from / to ProjectName
      run: sed -i 's/<base href="\/" \/>/<base href="\/%PROJECT_NAME%\/" \/>/g' dist/Web/wwwroot/index.html

    # Replace %PROJECT_NAME% in index.html with the Variable set above to match GitHub Pages repository subdirectory
    - name: Change base-tag in index.html from / to ProjectName
      run: sed -i "s/%PROJECT_NAME%/$PROJECT_NAME/g" dist/Web/wwwroot/index.html

    # copy index.html to 404.html to serve the same file when a file is not found
    - name: copy index.html to 404.html
      run: cp dist/Web/wwwroot/index.html dist/Web/wwwroot/404.html

    # Add .nojekyll so _framework will get loaded
    - name: Add .nojekyll
      run: touch dist/Web/wwwroot/.nojekyll

    - name: Commit wwwroot to GitHub Pages
      uses: JamesIves/github-pages-deploy-action@3.7.1
      with:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        BRANCH: gh-pages
        FOLDER: dist/Web/wwwroot
```

Make sure you replaced the PROJECT_NAME: your_project_name on line 4 in the main.yml file

Run the script below to deploy the update.
```bash
git add --all
git commit -m "Add Workflow CI/CD for github"
git push
```

This may take a moment to deploy. After deployment the following should occur:

- Refreshing the hosted site https://{user}.github.io/{project}/ should display the new blazor app
- Anytime a change is pushed to the "main" branch the pipeline should run, and the changes should appear.

### Run Locally
From VSCode in Bash
```bash
dotnet run --project "${PROJECT_NAME}/${PROJECT_NAME}.csproj"
```

From Visual Studio
- Open the solution file
- F5 to run with debugging
- Ctrl+F5 to run without debugging

### Beginner Tutorials
- [What is Blazor - YouTube](https://www.youtube.com/playlist?list=PLdo4fOcmZ0oUJCA3DCzKT79Oe3kdKEceX)
- [Intro - Rock Solid](https://www.rocksolidknowledge.com/articles/an-introduction-to-blazor-webassembly)
- [ezzylearning.net](https://www.ezzylearning.net/tutorial/a-beginners-guide-to-blazor-server-and-webassembly-applications)
- [Debugging With Hot Reload](https://dev.to/sacantrell/vs-code-and-blazor-wasm-debug-with-hot-reload-5317)
# BlazorWasmDeployOnly
