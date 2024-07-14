# Minecraft Plugin Runtime Test
Github action for testing minecraft plugins initialization during server load on different versions of paper server.

### How it works
#### Prerequisite Steps (Building the plugin)
1. Checkout repository (in the context of the action caller)
2. Download plugin build artifact from previous workflow build job
3. Set up Java 21
4. Setup Node 20
5. Execute index.js file
#### index.js (When this action is called)
1. Create and initialize ```eula.txt```
2. Fetch latest paper server build
3. Download paper server jar (server version from matrix variable as input data to this action) 
4. Execute mc server

### Usage
- Create an action file inside ```./github/workflows``` in the scope of your plugin repository and configure the steps if necessary:
```yml
name: Build with Maven and Do Runtime Test

on:
  workflow_dispatch:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
    
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4.1.7
      
    - name: Set up JDK 21
      uses: actions/setup-java@v4.2.1
      with:
        java-version: 21
        
# Choose you're Build-Tool
        
    - name: Gradle build
      run: gradle clean build
      
#    - name: Maven Build
#      run: mvn clean package --file pom.xml
      
    - name: Upload the artifact
      uses: actions/upload-artifact@v4.3.4
      with:
        name: artifact-${{ github.event.number }}
        path: 'build/libs/*.jar' # Change this according to the location and filename of your packaged jar, you may use wildcards
        # path: 'target/MyPluginJar*.jar'
  
  runtime-test:
    name: Plugin Runtime Test 
    needs: [build]
    runs-on: ubuntu-latest
    strategy:
      matrix:
        mcVersion: ['1.20.1', '1.20.2', '1.20.4', '1.20.5', '1.20.6', '1.21']
        javaVersion: ['21']
    
    steps:        
      - uses: sailex428/minecraft-plugin-runtime-test@v1.1.3 # specify action version, use latest as possible
        with:
          server-version: ${{ matrix.mcVersion }}
          java-version: ${{ matrix.javaVersion }}
          artifact-name: artifact-${{ github.event.number }}
```

### Plugins Included by Default During Runtime
- Slimefun

Suggestions are open for plugins that depends on other plugins. 
This will be based off from a resource file soon in order to accomomdate more plugins or so this repo can be forked to support your plugins.
