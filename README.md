# Overview

This repository is a space for collaboration and documentaion of the manifest tool and its output products. 

The manifest tool is used to collate scripts and configuration for a DESDL service binding. Its output is ingested by the ESDL engine to create a binding between an interface and a back-end, supplying the necessary configuration and runtime scripts.

Some of the terminology here needs a little cleanup, especially use of the terms script and transform.

## Files

In this directory are sample files showing how this could look. These files are also a suggestion for how the scripts and configurations could be stored in your repo.

| File | Notes |
|--|--|
| manifest.xml | The _main_ input file to the manifest tool that lists the inputs, search paths and output options |
| binding-config.xml | Based on the current binding configuration of DESDL services. Can contain deployment variables that the tool substitutes with actual values defined in **deploy-vars.xml** file|
| service-scripts.xml | Contains the service-level scripts potentially shared by all methods referenced in **binding-config.xml** | 
| deploy-vars.xml | Provides values for any of the deployment variables referenced in the **binding-config.xml** or any other input files |
| method-script-ppasearch.xml | Example scripts for the PPA method |
| method-scripts-berterniereport.xml | Example scripts for the BertErnieReport method |
| method-scripts-berterniesearch.xml | Example scripts for the BertErnieSearch method |
| manifest-out-binding-config.xml | Output of the manifest tool. Will be input to the ESDL engine to create and configure the binding and execute scripts (transforms) |

## Repository

Repository organization is left flexible. We'll provide an example and possibly some recommendations shortly.

# Roadmap

We'll implement the tool in several incremental phases to get some basic functionality relased now then improve subsequent releases based on our experience using it. Our intention is this will be used by DevOps to create a release package or container to hand off to the OPS team as immediately deployable.

## One

### Including Scripts
 
Based on Tim's feedback, instead of using `<xsdl:include/>` as originally proposed, change it so that it can be used to either include _all_ the scripts in the referenced file:

	<Binding>
		<Definition>
			<Script>
				<Transforms>
					<EspRequest>
						<xsdl:include-from file="service-scripts.xml"/>
						<!-- 
							By default all the transforms defined in service-scripts.xml
							are included here in this element
						-->
					</EspRequest>
				</Transforms>
			</Script>
		</Definition>	
	</Binding>

or just include some by name:

	<Binding>
		<Definition>
			<Script>
				<Transforms>
					<EspRequest>
 					  <xsdl:include file="service-scripts.xml" transform="GenericOutOfBand"/>
 					  <xsdl:include file="service-scripts.xml" transform="UserOutOfBand"/>
					</EspRequest>
				</Transforms>
			</Script>
		</Definition>	
	</Binding>

or as a list of names:

	<Binding>
		<Definition>
			<Script>
				<Transforms>
					<EspRequest>
 					  <xsdl:include file="service-scripts.xml" transforms="GenericOutOfBand,UserOutOfBand"/>
					</EspRequest>
				</Transforms>
			</Script>
		</Definition>	
	</Binding>

This way you can organize scripts in 'shared' files where different subsets of transforms are used by a wide variety of different bindings. Also you can put all the transforms you'd need for a given service or method in a single file and automatically include 

### Deployment Variables

As Tony pointed out the variable notation must be unique for each level of our scripts that does variable replacement. For deployment variables, ensure that the **%xxx%** notation that won't conflict with any other type of escaping or variable notation. 

### Outline

1. Implement as esdl sub-command
2. Take a manifest file as input, consisting of
    1. Search paths for includes
    2. Path to a deploy-vars file that maps variables to values for per-environment customizations (eg, Roxie URL)
    3. Path to service binding file
    4. Output specification- details on how and where the output is written
3. Parse given service binding file
		1. Search include paths to find transforms included
		2. Compile a binding-configuration including just the transforms/scripts explicitly referenced and found in the search paths
		3. Replace deploy-vars references with the values provided in the deploy-vars file
4. Output
    1. A binding-configuration file that has only referenced transforms included
		2. A log-transform file
    3. Possibly a shell script that would perform the binding

### Assumptions

1. The service-binding input is for the complete serivce and will replace any current service binding configurations
2. No validation is done to ensure all service methods have a configuration. Some deployments legitimately do not configure all methods in a service. Future behavior can give warnings for a method with no configuration.
3. Generate ERROR for name collisions found when resolving includes in a single scope: service or method-level for a single method, not between all methods.
4. The ESDL engine, including hte script engine, would only need to read in the **manifest-out-binding-config.xml** file and possibly a log-transforms.xslt file for the entire service. This may change based on Tony's config file refactoring.

## Two

1. Directly publish the changes to the binding, including logTransforms
2. Option to output single binding configuration file, including logTransforms

## Three

1. Keep deploy-vars a separate file
2. Update engine to treat deploy-vars as variable substitution
3. Update WsControl? to trigger deploy-vars update/read

## Four

1. Update UI to ingest & process this single file
2. Update UI with a method to ingest deploy-vars


## Five

1. Update a single method? 
2. How to handle with inherited transforms?

