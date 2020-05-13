# Thoughts	

Various thoughts and to-dos are listed here. Some are resolved and already included in the prototype.

1. How to configure for Proxied methods. Check example in HPCCSystems- /opt/HPCCSystems/examples or esdl examples.
2. Should the tool manage pushing out interface definitions and creating bindings also? I'm going to assume no for now
3. How do we tell where the transforms from an included script belong in the output - service vs. method level?
    1. Do we require the <include file="name"> to be located in the scope where it belongs? Then what if one file is included in both service and method- call it an error during manifest-time?
    2. Do we require the contents of each included file to have an attribute identifying its intended scope?
    3. Is there some way we could automatically determine this by where it's referenced?
4. Manifest should be a monolithic state for the entire service binding. We won't have a way to generate a manifest for a single service because:
    1. What about shared dependencies? Meaning- service level transforms it may call or inherit from.
    2. Can we maintain any guarantees about stability of overall service if we want to update just one method?
5. In the future add capability to update one + method, with these understandings:
    1. Method scripts may refer to existing service level scripts without changing them
    2. Maybe will be able to add new service level scripts. But even then would we need to do manifest-time name collision checking? What if a new service transform is the same name as another transform used in another method? 
6. Manifest file will need to have name of process, name of binding.
7. When parsing the included scripts
    1. store as text
    2. keep pointers to original source?
    3. keep pointers to where they are referenced?
    4. keep tabs on the transforms each contains as a list that has char* pointer & length & name instead of parsing each out into another separate object?
8. Create a dict for the deploy-vars and use that to see if vars in the service-binding need to be replaced.
9. Use xml namespaces for specifying/scoping included scripts. Use namespaces in file fragments, and also in final blob.
10. Extend esdl tool with manifest functionality?
11. Esdl tool + UI for publishing manifest
    1. Will the UI have a way to take the entire blob?
    2. Can the UI update separate sections or just the whole thing
12. Blob will contain what?
    1. all info currently in service binding?
    2. all info that is stored outside that scope too?
13. Separate file, referenced in manifest, that will include the per-environment values to substitute in. This would be the file OPS would switch out in each environment.
14. Manifest itself will just give paths to files to use, and those files will have 'includes' and namespaces to limit scope and uniquely identify which bits to include
15. What is syntax for manifest itself? Research other similar tools
    1. CMAKE
    2. ANT
    3. ecl
16. Could tool automatically generate some unique identifier for each statement that can be used in debugging output to identify where you are in the code.
17. What will a developers code-build-test cycle look like? Will the use of the manifest tool be required to build a configuration? Think about how we can make the cycle easy and quick.

## Binding/Blob Format

Tim's preferred solution is to just keep pTree. He will check each node 

1. if Child nodes then serialize in whatever order comes out and work with it
2. if a text node (then it is a serialized XML) assumption is that it's not wrapped in a parent (aka <Script>) node.
3. Tony wants people to be able to edit the config as XML
4. He doesn't want us to have any special attribute marker in the XML that marks something to be stored as blob
5. Tim thinks it should all be contained inside a <Script> element
6. All blobs will be contained in non-blob elements
7. Tim will go through pTree, if he has content (value aka blob) he'll assemble  document (preserving order) to parse with xpp
8. Current 'legacy' CustomRequestTransforms *could* have order preserved, but maybe not required.
9. Tim would not want the esdl_store to compile all the scripts for a service into a single object that he then needs to understand where all parts are from.
10. Otherwise he would want a way to query the esdl_store for the blobs at each level, but that is a pain