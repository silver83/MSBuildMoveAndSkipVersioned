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


This is implemented via MSBuild inside WebApplication1.csproj:

	  <!-- Start custom como template build step - move statics to versioned folder -->
	  <PropertyGroup>
		<AfterAddIisSettingAndFileContentsToSourceManifest>AddCustomSkipRules</AfterAddIisSettingAndFileContentsToSourceManifest>
	  </PropertyGroup>
	  <Target Name="AddCustomSkipRules">
		<Message Text="Adding Custom Skip Rules" />
		<ItemGroup>
		  <MsDeploySkipRules Include="SkipFilesInFilesFolder">
			<SkipAction>Delete</SkipAction>
			<ObjectName>filePath</ObjectName>
			<AbsolutePath>$(_DestinationContentPath)\\Public\\.*</AbsolutePath>
			<Apply>Destination</Apply>
		  </MsDeploySkipRules>
		  <MsDeploySkipRules Include="SkipFoldersInFilesFolders">
			<SkipAction>Delete</SkipAction>
			<ObjectName>dirPath</ObjectName>
			<AbsolutePath>$(_DestinationContentPath)\\Public\\.*</AbsolutePath>
			<Apply>Destination</Apply>
		  </MsDeploySkipRules>
		</ItemGroup>
	  </Target>
	  <Target Name="move_robots_in_msbuild" AfterTargets="PipelineCopyAllFilesToOneFolderForMsdeploy">
		<PropertyGroup>
		  <PackageRoot>obj\$(ConfigurationName)\Package\PackageTmp</PackageRoot>
		  <StaticsRoot>$(PackageRoot)\Statics</StaticsRoot>
		</PropertyGroup>
		<ItemGroup>
		  <Statics Include="$(StaticsRoot)\**\*.*" />
		  <StaticsFolder Include="$(StaticsRoot)" />
		</ItemGroup>
		<MakeDir Directories="$(PackageRoot)\Public\Statics\$(StaticsVersion)" />
		<Move SourceFiles="@(Statics)" DestinationFolder="$(PackageRoot)\Public\Statics\$(StaticsVersion)\%(Statics.RecursiveDir)" />
		<RemoveDir Directories="@(StaticsFolder)" />
	  </Target>
	  <!-- End custom como template build step -->