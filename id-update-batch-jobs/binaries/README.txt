==========================================
= Darkstar Version Update Batch (Beta)
==========================================

Table of Contents:

A) System Requirements
B) Building the Project
C) Preparing to Run the Batch
D) Batch Configuration File
E) Generate Npc Id Mapping File Batch
F) Npc Id Update Batch
G) Text Id Update Batch
H) Future Improvements
I) License Notices

==========================================
= A) System Requirements
==========================================

Build Requirements:
1) Apache Maven: http://maven.apache.org/download.cgi
2) Java 7 JDK: http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html
3) Optional: Eclipse w/ Maven Plugin

Runtime Requirements:
1) Java 7 JRE: http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html
2) POLUtils/MassExtractor: https://github.com/Windower/POLUtils/tree/master/Binaries

==========================================
= B) Building the Project
==========================================

The Darkstar Batch Jobs are written in Java, and are built with Apache Maven, an industry standard
build system for Java projects. In order to build the project, you will need both Apache Maven and
the Java 7 JDK installed on your machine.

Building from the Command Line:

1) cd to <Darkstar_Root>/batch/source/darkstar-batch/
2) Run "mvn package"
3) The binary jar will be built and placed in the "target" directory.

Building from Eclipse:

1) Do File -> Import
2) Select Maven -> Existing Maven Projects
3) Point Eclipse to "<Darkstar_Root>/batch/source/darkstar-batch/"
4) Import the Project
5) Right Click and say "Run as Maven Build"
6) Type "package" as your goal
7) Click "Run".
8) The binary jar will be placed in the "target" directory within the project.

==========================================
= C) Preparing to Run the Batch
==========================================

In order to run the batch jobs, you must run the POLUtils Mass Extractor, and
point the batches to a complete dump. This guide will assume you have POLUtils
installed in the default location, and FFXI on the computer you are running on.
See system requirements for a link to POLUtils.

1) Click the Start Button and in the search window, type "cmd" and press enter.
2) The command prompt should open.
3) cd\Program Files\Pebbles\POLUtils
4) Run: MassExtractor.exe "<Darkstar_Root>/batch/binaries/polutils_dump/"
Fill in <Darkstar_Root> with the root path of your Darkstar Clone.
5) Ignore any warnings from the output. You should have a bunch of XML files
in your polutils_dump folder references above. If you do, you are ready to run
the batch jobs

NOTE: You may or may not need to run MassExtractor.exe as admin if you are a limited
user with UAC enabled on your computer. I'm not sure.

==========================================
= D) Batch Configuration File
==========================================

The section refers to the file "config.properties", in the binaries folder:
"<Darkstar_Root>/batch/binaries/polutils_dump/config.properties"

polutils_dump - Sets the path to the POLUtils Dump. 
mapping_file - NPC Id Mapping File. Generated by the Generate NPC Id Mapping File Job.
darkstar_root - Path to the Root of your Darkstar Git Clone
minZoneId - Zone ID to Start Executing at
maxZoneId - Zone ID to End Execution at
npcIdMarkGuesses - Mark FIXME: on Guesses in the NPC Id Job (Note: toggling this may not work yet)
textIdMarkGuesses - Mark FIXME: on Guesses in the Text Id Job

Section: # Npc Ids - Mob List Mappings - Comment Out Zones You Don't Want to Run
Maps Mob List Files from POLUtils Dump to a Zone Name as Spelled in the Comments 
in the "npc_list.sql" file (THIS MUST MATCH). To make the batch aware of new zones,
add mappings here. To stop the NPC ID batches from running for specific zones, 
comment them out with a # sign here.

Section: #Zone ID Mappings
Maps Zone IDs to their folder in Darkstar. Used by the Text ID Update Batch. To
stop this batch from running for specific zones, comment them out with a # sign
here.

==========================================
= E) Generate Npc Id Mapping File Batch
==========================================

Batch Job to Generate a Mapping File Between Darkstar SQL & POLUtils
 
For this to work properly in its entirety, we must be in a known good state.
 
This is necessary in order to be able to attempt to automate NPCs where our name
in the SQL does not match the name given in POLUtils.
 
Once we have a proper mapping, it will allow the other batch jobs to autofix these
cases
  
If a mapping is bad because we are currently in a bad state, it will not be able to
fix the problem, but should not make it worse either.
  
As we manually fix these cases in the npc_list, we should rerun this job to regenerate
the batch file until we are in a completely good state.
 
Once a good mapping has been established for a given entry, the NPC ID Update job should
be able to auto-fix that NPC going forward.
 
To run the job:
1) Ensure you have a configured a POLUtils Dump **FOR A VERSION CURRENTLY IN SYNC WITH 
THE NPC_LIST SQL**
 
NOTE: This is VITAL or you will mess up a lot of NPCs with the NPC ID Update Batch.
Because of our desync in naming, we need this mapping file for a lot of NPCs. But
the mapping file MUST BE GENERATED WHEN WE ARE IN A KNOWN GOOD STATE
 
So if you are generating a new mapping file, generate this with a POLUtils Dump 
from the Old Version  BEFORE updating. Then use a POLUtils Dump for the NEW VERSION 
with the Npc Id Update Job.
 
If you are updating the mapping file after manual fixes, you may use a dump from the
current version

2) Run GenerateNpcMappingFile.bat. This will overwrite "npcid-mapping.properties"

3) See "darkstar-batch.log" for details of what happened. If you need more info, you
may enable debug logging by editing "log4j.properties" and changing "INFO" to "DEBUG" at
the top and rerunning the batch.

REMEMBER: This file is only as good as Darkstar's current state. Npcs currently broken
by ID problems will be broken in the mapping, and thus still broken after the batch update.
It should not make them any worse though.

==========================================
= F) Npc Id Update Batch
==========================================

Batch Job to Update Npc Updates for Darkstar after a Version Update

This Job requires both a POLUtils Dump and a Npc Id Mapping File for NPCs
where the naming between Darkstar SQL & POLUtils is different.
 
For these npcs, accuracy will depend on accuracy of the mapping file. If the mapping
file is generated right before the version update, it should keep things from getting worse,
but may not auto-fix everything if mapped entries are bad.
 
After such cases are manually fixed, the mapping file should be regenerated, and they should
then be auto-fixable in the future.

1) Ensure "npcid-mapping.properties" exists in your working directory. See previous section
for details and you have a current POLUtils Dump.

2) Run "NpcIdUpdate.bat". This will make updates to the "npc_list.sql" file in your SQL directory.

3) Review the results carefully with a diff tool. See the following limitations:

   a) NPCs incorrectly mapped in the mapping file will be incorrectly updated. This means they were
      already broken though, and won't be worse off than before.

   b) Check carefully NPCs for which there are multiple of the same name. These are subject to being
      wrong in the current version of the batch. Sometimes this is because of us not being consistent
      with naming. Other times it is due to IDs bumping down one after a large gap, in which case the
      batch cannot always detect this yet. But it should be obvious with this happens in the diff.
      
4) After manually reviewing the diff changes and making fixes as necessary, the file may be committed.

5) See "darkstar-batch.log" for details of what happened. If you need more info, you
may enable debug logging by editing "log4j.properties" and changing "INFO" to "DEBUG" at
the top and rerunning the batch.

IMPROVING THE BATCH:

Two main things can improve this batch easily:

1) Naming NPCs consistent with POLUtils wherever possible (easiest long term fix).

2) Manually fixing broken NPCs and regenerating the mapping file.

==========================================
= G) Text Id Update Batch
==========================================

Job to Perform a Batch Update of Text IDs after a Version Update

Requires a POLUtils Dump of dialog tables for reference. This job
makes comparisons between the comment in the TextID file and the
dialog tables in POLUtils to find the updated Text ID

Accuracy varies with two factors:

1) Accuracy of the comment in the TextID files. Typos will break the
comparison

2) State of bad chars & etc. in the dialog table dumps. This job will
try to strip them out of both sides for comparison, but that may leave
the sting without enough context to make it unique. In this case, its
possible the wrong ID would be selected.

Ids not concretely matched can be prepended with FIXME: for manual review.

Please check for these after the job and fix them before committing.

To run the batch:

1) Ensure you have a current POLUtils Dump

2) Run TextIdUpdate.bat

3) Check "darkstar-batch.log" - the last section will tell you any files with FIXMEs

4) Manually review and fix all the FIXMEs, comparing with POLUtils manually.

5) When fixing FIXMEs, update the comment in the LUA file with the correct POLUtils 
text if possible to make it auto-fix next time.

6) After all FIXMEs are fixed manually, the file can be committed

7) See "darkstar-batch.log" for details of what happened. If you need more info, you
may enable debug logging by editing "log4j.properties" and changing "INFO" to "DEBUG" at
the top and rerunning the batch.

IMPROVING THE BATCH:

Two main things can improve this batch easily:

1) Improving the comments in the LUA file to more closely match POLUtils.

2) Improve the Bad Char REGEX Filters in "badcharacters.dat" in the project source (or fix POLUtils :p)

==========================================
= H) Future Improvements
==========================================

Other than bugfixes, I intend to add a module for updating NPC IDs in Scripts, using data collected
from the NpcIdUpdate job. The accuracy of this task would vary depending on how in sync the ids in
the scripts and the npc_list SQL were at the time the batch jobs were run. Similarly to the NpcIdUpdate
job's situation with the mapping file, anything out of sync before the jobs were ran would still end up
out of sync, but anything that was in sync would get auto fixed.

Look forward to this in a future update.

==========================================
= I) License Notices
==========================================

Dark Star Batch:
As noted in all source files, this batch project is distributed under the GNU GPL v3 (or higher), as
is the case with the core Dark Star Server.
http://www.gnu.org/copyleft/gpl.html

Apache Log4j, Apache Commons Lang, Apache Commons IO:
These dependencies are distributed under the Apache License V2.0
http://www.apache.org/licenses/LICENSE-2.0

Both GNU & the Apache Software Foundation consider Apache License 2.0 & GPL v3 Compatible w/ Each other,
so long as the final project is distrubted as GPL v3.

https://www.gnu.org/licenses/license-list.html
http://www.apache.org/licenses/GPL-compatibility.html