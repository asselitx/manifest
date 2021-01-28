# esdl Tool

The esdl tool aids with creating and managing ESDL-based and Dynamic ESDL services on an HPCC cluster. It consists of several different commands.

## Overall notes for Tim and Tony

Here is my overview of the design direction:

1. Think of the manifest as config-as-code. The operator using the manifest to generate output shouldn't need to know which options to pass on the command line.
2. Collapse the template file into the manifest.
3. Think of the template as having two kinds of markup
    1. operations- Like `<include>` which will have an associated namespace and an explicit prefix.
    2. boilerplate- Like `<Method>` which gives structure to the output. These elements will most often use the defaut 'no prefix'.
4. Think of even the boilerplate parts of the manifest like a programming language- it should not mirror the output exactly, but provide a similar, simplified structure. 
    1. This isn't quite the generic 'copy-through' behavior Tim and I were envisioning, but it will simplify the boilerplate. The tightened coupling between the ESDL implementation and the tool does not offset this benefit.
    2. We can add markup in the future to get generic copy-through behavior.
    3. The tool will augment the output in specific ways based on certain boilerplate markup. For instance `<Scripts>` will wrap its contents in a CDATA section.
5. By default, generate 8.0 compatible output, later add markup or options to target older versions.
6. In the future we can add validation for `<include` operations. I'll open a Jira and we can discuss if we should validate each include as it's read in, or if schema validation on the final output is preferable.
7. Change the command name to `build-config` since it will be building bundles and other output in addition to bindings.

## build-config

Creates an xml config file for an ESDL service or a component of a service deployment. Currently two forms of config output are supported- the *bundle* and the *binding*. The *bundle* is a self-contained dynamic service configuration. It is suitable for configuring deployments to containerized cloud environments. The *binding* is a legacy service configuration format designed primarily for basic bare-metal deployments. 

This command takes a manifest file as input. The manifest file includes options and operations to generate the output, along with template markup that dictates the general format of the output.

### Manifest file

The format of a manifest file is below. Future updates will add more capabilities.

    <em:esdl-manifest xmlns:em="urn:hpcc:esdl:manifest">
        <em:template>
        </em:template>
        <em:output write-file="1">
            <em:file path="/full/path/to/output/ws_foobar-binding.xml"/>
        </em:output>
    </em:esdl-manifest>

### Bundle Template

This version of the tool supports generating these elements of a bundle:

1. Enumeration of all methods in the service
2. Configuration of the back-end, gateways and other options of each method
3. Transform scripts defined in the scope of the service
4. Transform scripts defined in the scope of each method
5. ESDL interface definition of the service

Organize your scripts as you see fit in a repo, then use `<include>` tags in the template to include the source of the referenced scripts in the configuration output. The format of a bundle template looks like:
    
    <em:template>
        <ESDLBundle>
            <Binding>
                <Definition> <!-- keep as per Tim's comment re: @auth_feature? -->
                    <Methods>
                        <Scripts>
                            <em:include file="/full/path/to/FooBarService-script1.xml"/>
                            <em:include file="/full/path/to/FooBarService-script2.xml"/>
                        </Scripts>
                        <Method name="FooBarReport">
                            <Gateways>
                                <Gateway name="baz" url="www.baz.com/api"/>
                            </Gateways>
                            <Scripts>
                                <em:include file="/full/path/to/FooBarReport-script1.xml"/>
                            </Scripts>
                        </Method>
                    </Methods>
                </Definition>
            </Binding>
            <Definitions>
                <em:include file="/full/path/to/ws_foobar.ecm" service="WsFooBar" searchPaths="/path/one:/path/one/two"/>
            </Definitions>
        </ESDLBundle>
    </em:template>

#### Output

The generated configuration is of this form:

    <ESDLBundle>
        <Binding>
            <Definition>
                <Methods>
                    <Scripts>
                        <![CDATA[<Transforms>
                            <es:CustomRequestTransform xmlns:es="urn:hpcc:esdl:script" name="script1">...</CustomRequestTransform>
                        <es:CustomRequestTransform xmlns:es="urn:hpcc:esdl:script" name="script2">...</CustomRequestTransform>
                        </Transforms>]]>
                    </Scripts>
                <Method>
                </Method name="FooBarReport">
                    <Scripts>
                        <![CDATA[<Transforms>
                            <es:CustomRequestTransform xmlns:es="urn:hpcc:esdl:script" name="report-script1">...</CustomRequestTransform>
                        </Transforms>]]>
                        <Gateways>
                            <Gateway name="baz" url="www.baz.com/api"/>
                        </Gateways>
                    </Scripts>
                <Methods>
            </Definition>
        </Binding>
        <Definitions>
            <![CDATA[<esxdl name="WsFooBar">...</esxdl>]]>
        </Definitions>
    <ESDLBundle>


### Binding Template

The format of a binding template is below.

    <em:template>
        <Binding>
            <Definition> <!-- keep as per Tim's comment re: @auth_feature? -->
                <Methods>
                    <Scripts>
                        <em:include file="/full/path/to/FooBarService-script1.xml"/>
                        <em:include file="/full/path/to/FooBarService-script2.xml"/>
                    </Scripts>
                    <Method name="FooBarReport">
                        <Gateways>
                            <Gateway name="baz" url="www.baz.com/api"/>
                        </Gateways>
                        <Scripts>
                            <em:include file="/full/path/to/FooBarReport-script1.xml"/>
                        </Scripts>
                    </Method>
                </Methods>
            </Definition>
        </Binding>
    </em:template>

#### Configuration Output

    <Binding>
        <Definition>
            <Methods>
                <Scripts>
                    <![CDATA[<Transforms>
                        <es:CustomRequestTransform xmlns:es="urn:hpcc:esdl:script" name="script1">...</CustomRequestTransform>
                    <es:CustomRequestTransform xmlns:es="urn:hpcc:esdl:script" name="script2">...</CustomRequestTransform>
                    </Transforms>]]>
                </Scripts>
            <Method>
            </Method name="FooBarReport">
                <Scripts>
                    <![CDATA[<Transforms>
                        <es:CustomRequestTransform xmlns:es="urn:hpcc:esdl:script" name="report-script1">...</CustomRequestTransform>
                    </Transforms>]]>
                    <Gateways>
                        <Gateway name="baz" url="www.baz.com/api"/>
                    </Gateways>
                </Scripts>
            <Methods>
        </Definition>
    </Binding>

### Usage

*Note for Tim and Tony: the `--manifest` option should actually be a required positional argument, I'll make that change with any other changes you suggest*

    Usage:
    esdl build-binding [options]

    Options:
        --manifest <file>   Path to manifest file.
        --no-cdata          Output binding without wrapping scripts in CDATA.
        -I | --include-path <path>
                            Search path for includes referenced in the source file. Can
                            be used multiple times.
        --help              Display usage information for the given command
        -v,--verbose        Output additional tracing information
        -tcat,--trace-category <flags>
                            Control which debug messages are output; a case-insensitive
                            comma-delimited combination of:
                                dev: all output for the developer audience
                                admin: all output for the operator audience
                                user: all output for the user audience
                                err: all error output
                                warn: all warning output
                                prog: all progress output
                                info: all info output
                            Errors and warnings are enabled by default if not verbose,
                            and all are enabled when verbose. Use an empty <flags>
                            value to disable all.

To target platform versions less than version 8, use the --no-cdata flag:

`esdl build-binding --no-cdata --manifest <path/to/manifest.xml>`

Note that the `-I | --include-path` option is not yet implemented. All script include paths in the template must be absolute.
