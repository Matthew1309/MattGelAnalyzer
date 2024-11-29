How to use this project. 
1. `git clone` to a folder
2. Locate your Fiji installation and navigate to the jars/ folder
3. Replace the existing `ij-1.54f.jar` with the one in this project.
4. Before selecting gel lanes do `Analyze -> Gels -> Gel options` and change the path and filename to what you want.
5. Proceed as usual (select first lane, next lane, plot lanes)

### Setup
1. On an ubuntu linux machine, download Eclipse following the advice from https://imagej.net/develop/eclipse. 
2. Went here https://www.azul.com/downloads/?version=java-8-lts&package=jdk-fx#zulu and downloaded the linux x86 64-bit JDK FX
3. Then I went here and got Eclipse https://www.eclipse.org/downloads/. 
4. It put the program into `/home/something/something/eclipse` rather than `home/eclipse`, but the program runner worked fine and opened up eclipse without issues.
5. Then from their respecitive github and download site I got both the raw ImageJ source code and Fiji onto the linux machine. Did git clone https://github.com/imagej/ImageJ.git and went https://imagej.net/software/fiji/ to get the linux version of fiji. 
After quite a bit of playing around, I noticed that the fiji had a folder named jars/ inside of which was a zipped and compiled java file called `ij-1.54f.jar`. On linux if we double click it we can see inside this zip, and see that it contains the code we want to edit, `GelAnalyzer.class`. But we cannot edit it here, rather we need to edit it while it is a java file in the ImageJ source code, compile it, then put that file in.

### Editing ImageJ source code
I'm still not sure how the files know where to appear on the ImageJ dashboard, but that is a mystery for a better programmer. 
1. With Eclipse open we can do `File -> Import -> Existing Maven Projects` click next, then browse to the ImageJ (not fiji) folder and click Finish. This will import ImageJ code into the package explorer on the left side of my screen. 
2. At this point you could click the run/compile button and the classic ImageJ viewer will open up, and it has all the functionality you need!
3. To actually edit the code we go into the package `ij -> plugin -> GelAnalyzer.java` and change the code to the `GelAnalyzer.java` I have in this project. 
4. Here I added a button to the GelAnalyzer options whether to save data to a file or not, as well as a section to add the path and filename to save to. It isn't the nicest and sleekest, but it gets the job done. I also populate a `ResultsTable` object (ImageJ thing) with the data stored in profiles and use the built-in save functionality.
5. At this point, running the ImageJ in Eclipse works as expected and saves my gel data!

### Updating Fiji
1. We now need to compile our ImageJ code into a `.jar` file.
2. In Eclipse go to `Project -> Properties -> Java compiler`. You need to uncheck some box, then set the Compiler to version 1.8 instead of Java 9 (default). Click finish.
3. Right click the package explorer and select `Export -> Java -> Jar`
4. Then select the entire ij tree, manifests, macros, etc and select okay. Name the file something like `ij-matt.jar`
5. Now I don't know how to appropriately produce a plugin that has its own menu, so what I did was instead of putting this jar file into `Fiji/plugins` or `Fiji/jars` folders, I double clicked the `ij-matt.jar` and double clicked the `ij-1.54f.jar` then deleted the actual `GelAnalyzer.class` file and drag and dropped mine in. TaDa! It now works on linux.
6. I tested this on Windows by deleting my naitive `ij-1.54f.jar` and replacing it with my augmented one I have in this project and it works as expected!


### `GelAnalyzer.java` changelog
I added the following class variables to give users the option of saving the data profiels and where to save them.
```
// matt addition
	static String filepath = "/home/matt/Downloads/gel_lanes.csv";
	static boolean writeToFilePath = true;	
```
I edited the `showDialog()` function
```
// matt addition
		gd.addStringField("File path: ", filepath);
		gd.addCheckbox("Write to file", writeToFilePath);
		filepath = gd.getNextString();
		writeToFilePath = gd.getNextBoolean();
```
I edited the `plotLanes()` function by populating a ResultsTable ImageJ object with the results in our profiles, then saving the table to the user specified path if they set the option in the GelAnalyzer options.
```
// Write profiles to a file (Matt/ChatGPT edit) here here
		if (writeToFilePath) {
			ResultsTable rt = new ResultsTable();
			for (int i = 1; i <= nLanes; i++) {
			    double[] profile = profiles[i];
			    for (int j = 0; j < profile.length; j++) {
			        // Ensure that the table has enough rows
			        if (j >= rt.size()) {
			        	rt.incrementCounter(); // Adds a new row
			        }
			        // Add the profile value to the column corresponding to the lane
			        rt.setValue("Lane " + i, j, profile[j]);
			    }
			}
			rt.save(filepath);
		}
```



