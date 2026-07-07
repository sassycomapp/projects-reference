All the projects that we create are anvil.works projects, They will all use the Anvil Material 3 theme And Dependency components when they will all use the MUI overlay . therefore many aspects which are common to all projects will be saved the "C:\projects\project-library-global" And can be referenced from all projects .
All files and folders which are canonical to a Specific  project are saved in that Project project-library 
all Anvil based projects Two repos: The code repo and the documentation repo. The documentation repo is always called project Library
All project repos including  C:\projects are registered As sources on G Brain and configured with opencode, gstack, github, matt pocock skills
Project folders only hold documentation which is durable and canonical to the specific project , everything that is general or global Lives in the "C:\projects" folder

Project files prefixes :
"spec-" = All specifications files
"policy-" = all policy documents
"ADR-" = all ADR Documents
(No prefix)  = canonical durable project documents
"screen-" = All screens (html docs)
"wireframe" - = All wireframes (html docs)
"tmp-" = all Template documents
"chk-" = All checklist documents
"sop-" = All standard operating procedures 

"C:\projects\project-library-global"
All the folders which hold files which can be shared across all projects. these folders have the suffix: "-global". These files are maintained in the central repository for ease of maintenance and updating so that we do not end up with different copies of the same files 
"C:\projects\project-library-global\adr-global"
"C:\projects\project-library-global\checklists-global"
"C:\projects\project-library-global\docs-standard-global"
"C:\projects\project-library-global\policy-global"
"C:\projects\project-library-global\specifications-global"
"C:\projects\project-library-global\templates-global"

docs-standard-global:
These are standard Saas / dev documents.The format in these files is not necessarily intended to be used as is in the projects . It is an indication of the documents that will be required in the project and an example of what should go into but not definitively 

Project-template:
This is currently a project in development under the title "C:\projects\dev-project-template". . It is intended to kick start a new project . the developer will copy in the project-template into the new project and this folder will become the project library for the new project  

Standard Operating Procedures:
These are predetermined M.O.'s (Modus Operandi). These files are intended to constitute the standard method of executing a specific task . one task per file. each Standard operating procedure file has a prefix  "sop-"

"C:\projects\dev-project-template"
When the dev-project-template project is complete The project template will move into the "C:\projects\project-library-global" folder 

When any dev project has been completed and has been deployed, The project is live , In the dev project gets archived . It'll no longer be directly in the "C:\projects" folder but will move to: "C:\projects\deployed-projects"

Reusable documents :
No project will use all the ADR global documents; all the Checklists; all the policy global; all the specifications global; all the SOP global; all the templates global; etc.. The individual projects will reference the specific documents in those folders that pertain to them .  

State of global documents:
All the documents in "C:\projects\project-library-global" They have been hurriedly and haphazardly assembled Many have been copied in directly from other project folders The documents do need to updated and in some cases created .the documents are the prefix "mt-" or empty documents which require to be written.  Some folders only have a single document in which is really just a placeholder for the documents that must be created in that folder 

PDLF runs
It is possible Two or more projects can be in a state of development in the PDLF at anyone time . PDLF must manage this. do not get lines crossed between the projects 