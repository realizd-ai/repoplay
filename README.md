# repoplay
macOS CLI tool for applying the changes staged by the [RepoPrompt](https://repoprompt.com/) application.

## RepoPrompt CoT Prompt
This prompt is intented to work with the **RepoPrompt** macOS application.

The `RepoPrompt CoT Prompt.txt` prompt should be saved as a stored prompt within **RepoPrompt**. Select it as the only stored prompt to apply to your instructions. Once the **RepoPrompt** prompt bundle has been copied and given to the LLM of your choice, it will produce output in this XML format.

## XML format
The output of the LLM will be in the form of a single XML object in the format below. It supports basic operations for modifying a repo: create, patch, delete, move, dup.

```xml
<operations>
  <!-- Creates a new file -->
  <operation type="create">
    <destinationPath>/absolute-path/ContactForm.tsx</destinationPath>
    <content><![CDATA[
import React from 'react';
export default function ContactForm() { return <form>...</form>; }
    ]]></content>
  </operation>

  <!-- Applies a patch (unified diff) to an existing file -->
  <operation type="patch">
    <sourcePath>/absolute-path/Header.tsx</sourcePath>
    <diff><![CDATA[
@@ -2,3 +2,3 @@
 set option1
-set option2
+set option3
 set option4
    ]]></diff>
  </operation>

  <!-- Deletes an existing file -->
  <operation type="delete">
    <sourcePath>/absolute-path/OldComponent.tsx</sourcePath>
  </operation>

  <!-- Moves or renames a file -->
  <operation type="move">
    <sourcePath>/absolute-path/LegacyButton.tsx</sourcePath>
    <destinationPath>/absolute-path/ui/LegacyButton.tsx</destinationPath>
  </operation>

  <!-- Duplicates an existing file -->
  <operation type="dup">
    <sourcePath>/absolute-path/src/components/Template.tsx</sourcePath>
    <destinationPath>/absolute-path/src/components/TemplateCopy.tsx</destinationPath>
  </operation>
</operations>
```

The XML output the LLM produces should be saved to a file.

### Reduced tokens

Most changes will be patches to existing files. For these, the XML file uses _unified diff_ format in order to minimise output token counts and reduce the time taken to generate the output.

## repoplay command

The `repoplay` bash script can then be used to apply those changes. The paths inside the XML are absolute paths so it doesn't matter where you are when you play the changes.

```
$ ./repoplay -h                                                                                             ✔
Usage: repoplay [OPTION]... XML-PATCH-FILE
Apply repository changes from an XML file.

Options:
  -h, --help       Show this help message and exit
  -x, --xmlhelp    Show the XML format of the patch file
  -v, --verbose    Enable verbose output
  -d, --dry-run    Validate input and show planned actions without modifying files

Example:
  repoplay changes.xml
  repoplay -v -d changes.xml
  repoplay --dry-run changes.xml
```

## Safety

The script is making source code changes, so it is cautious. It prechecks the XML and the files it will operate on. If it sees any problems, it exits before making changes. It backs up the files that it intends to modify and — if it later encounters a problem — the original files are restored from these backups.

There's also a dry-run option which shows the changes that would be made without making them.
