# The 360Giving Extensions Registry

Welcome to the 360Giving Extensions Registry

* The `extensions/` folder contains the registry entry files representing extensions which have been added to the registry.
  * The names of the registry entry files determine the code which should be declared for this file in 360Giving data. For example `abc.json` would be declared as `"abc"` in the `extensions` array of 360Giving package metadata.
* The `schema/` folder contains a schema for validating and quality-control of registry entry files.
* The `examples/` folder contains an example of a registry entry file to copy and use, and should also be used to test any updates to the registry schema.

## Registry Maintenance

This section contains information on topics related to maintaining this registry. It is intended for members of the 360Giving Team and their technical partners.

### Governance

The [360Giving Governance Process](https://standard.threesixtygiving.org/en/latest/about/governance/#) approves new extensions to 360Giving. This means that the *contents* of the registry is governed. All new extensions must be approved by the 360Giving Stewardship Committee before being added to the registry via e.g. a Pull Request.

The *structure and features* of the registry may evolve over time. In practice, we do not expect them to, however the 360Giving Team can revise the registry structure to accommodate evolving needs of the 360Giving publisher and user community. This could include:

* changing the folder and file structure
* changing the registry-entry json schema
* adding or removing additional automated tests
* adding or editing documenation and guidance for using the registry

### Adding new registry entries

All new extensions should be explicitly approved by the 360Giving Stewardship Committee (SC). Once the SC is happy that the extension may be developed and/or included in the registry and the extension has been thoroughly tested &ndash; create a registry entry file under `extensions/` and raise a Pull Request for review.

### Updating compatibility information

The 360Giving schema will continue to develop outside of the rhythms of individual extensions. Whenever the 360Giving schema undergoes a MINOR or a MAJOR release, the extensions in the registry should be tested for compatibility against this new version. This should be performed by a member of the 360Giving team or technical partners.

These tests are not automated, but care should be taken over:

* are there any conflicting property names or definition names?
* are there any conflicting validation requirements?
* are there any conflicting codelists?

Once an extension has been reviewed as being "compatible" with a particular version of 360Giving, update the registry entry's `compatibility` array to include a string representing the version of 360Giving it is known to be compatible with e.g. `1.1` or `1.2`.

Please note: the 360Giving Extensions mechanism was added in version `1.4` of 360Giving, but technically speaking the `extensions` array could be added by publishers using version `1.1` onwards. Values below `1.1` should throw an error when checked against the registry entry schema. Only MINOR versions should be considered, as PATCH updates shouldn't change the 360Giving schema in any way.

### Changing the Registry Entry schema

All registry entry files must conform to the schema located at `schema/registry-entry.json`. Edit this file to change the registry schema e.g. add new fields, change titles and descriptions etc.

Whenever doing this, please do the following due diligence:

* check `schema/registry-entry.json` for JSON formatting errors
* check that `schema/registry-entry.json` is conformant against the [JSON Schema Metaschema (draft 2020-12)](http://json-schema.org/specification.html#meta-schemas)
* ensure that all extension entry files under `extensions/` are conformant to the edited schema, so that the automated tests don't break

## For Tool Developers

This section is intended for those who are developing tools for 360Giving and want to be able to use this extensions registry to find extensions declared in publisher data to create an extended schema for interfaces, validation, etc.

One of the goals of the registry is to provide tools with a way to find extension schema files and retrieve them, as well as provide tools with some additional metadata about the extension should they wish to use it or display it in an interface.

The design of the registry should hopefully make this as straightforward as possible, and will essentially involve retrieving a JSON file from the registry and using its contents to further collect the extension schema and codelist files.

Extensions in the registry are represented by JSON files and are stored in the `extensions/` folder. The job of tools is to match extension codes to registry entry files in this folder. This file will then provide information on where to retrieve the extension schemas, as well as additional metadata. The base URL of this folder is `https://raw.githubusercontent.com/ThreeSixtyGiving/extensions-registry/main/extensions/`. Tools should store this to construct URLs for retrieving registry entries.

### Steps

Tools should check for the presence of the `extensions` array in the 360Giving Package Metadata. If it is present do the following for each string in the file:

1. Append the string + `.json` to the extensions folder base url given above. For example if the string is `abc` you will end up with `https://raw.githubusercontent.com/ThreeSixtyGiving/extensions-registry/main/extensions/abc.json`
2. Retrieve the file at this URL and parse it. If you receive a `404` response, then the extension is not in the registry and an appropriate error message should be presented.
3. Once the file is parsed, check for the presence of the `dependsOn` array.
  * If `dependsOn` is present, for each string in the array perform this stepwise process from Step 1 recursively until no further dependencies remain. Then proceed to Step 4 for this extension.
  * If `dependsOn` is not present, proceed to Step 4.
4. For each entry in the `schemas` array perform the following:
  * Take the URI stored in the `extensionSchema` field and retrieve the file at this URI. Create appropriate error messages in the event of HTTP 40x and 50x error codes.
  * Perform a JSON Schema merge patch of the extension schema file onto either the 360Giving Schema or the 360Giving package schema as indicated by the value of `target`.
5. Check for the presence of the `codelists` array in the registry entry file.
  * If it is present, each of the values in the array will be a URI to a CSV file. Retrieve the file stored at this URI and store it in an appropriate place to later compile the schema.
  * If the codelist would replace or overwrite an existing codelist from the 360Giving Standard, do NOT save the file and report an appropriate error
6. Once all of the schema patches have been applied and the codelists have been retrieved, generated a compiled schema from the extended schema and codelists.
7. Validate the 360Giving data file using the new extended schema.

Additional considerations:

* The `compatibility` array indicates whether a particular version of 360Giving has been tested for compatibility for this extension. If value of the data file's `version` property does not match a value in the `compatibility` array, tools may decide whether to discontinue the process early and display an appropriate error message.
* The behaviour around codelists may change in the future, depending on the goals of 360Giving and the needs of publishers.

## Submitting an extension

All extensions must be explicitly approved by the 360Giving Stewardship Committee. They may then be included in the registry as part of actioning this approval.

If you want to discuss an extension for the 360Giving Data Standard please get in touch! Email `support@threesixtygiving.org` to contact the team.

### Extension Repository Structure

The 360Giving Extensions Registry does the heavy lifting of supporting tools to find the extension schema files, but there are a few expectations of extension repositories.

1. All extension schema files must be conformant against [JSON Schema Draft 04 Metaschema](http://json-schema.org/specification-links.html#draft-4), as this is the version used by the 360Giving Standard.
2. All extension schema files must live under a `schema/` folder which is located directly beneath the base URL of the extension repository or the base URL given in the value of `extensionBaseUrl`. For example if your base URL was `https://example.org/my-extension` then all extension schema files must be located underneath `https://example.org/my-extension/schemas/`.
3. Similarly, all extension codelist files must live undeneath a `codelists/` folder directly beneath the base URL of the extension repository or the base URL given in the value of `extensionBaseUrl`. All codelist files must be valid csv format files and cannot have a filename which matches a codelist file located [in the 360Giving standard repository](https://github.com/ThreeSixtyGiving/standard/tree/master/codelists).
