Configure and deploy the iis site once manually and deploy once via "import from package"

Run these:
msbuild  .\WebApplication1.csproj /p:VisualStudioVersion=11.0 /p:PublishProfile=Test /p:DeployOnBuild=true /p:StaticsVersion=4.0
Packages\Test\Package.deploy.cmd /y
msbuild  .\WebApplication1.csproj /p:VisualStudioVersion=11.0 /p:PublishProfile=Test /p:DeployOnBuild=true /p:StaticsVersion=3.0
Packages\Test\Package.deploy.cmd /y


In the source folder you have 
`Statics\[some files]`

After packaging with a /p:StaticsVersion=5.0 and publishing you'll get 
`Statics\5.0\[some files]`

and any old version of Statics folder will not be overrun during publish.

This is mainly to allow versioned resource urls, mixed with unversioned api.
