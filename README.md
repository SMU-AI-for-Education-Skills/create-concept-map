## Purpose
Guides the process of creating concept maps out of text(s) provided by the user. Concept maps are generated as standalone HTML files, with options to produce fully fleshed-out maps as supplementary material, or deliberately including gaps and blanks as learning activities. To aid learning, maps are interactive and editable, and include the option to save the edited versions.

## Using the skill 
You may download the latest version of this skill as a ZIP file under `Releases` on the right of this Github Repository. You may refer to our guide to learn how to [use this skill within your AI tools](https://github.com/SMU-AI-for-Education-Skills#accessing-and-using-skills). 

## How this works
Takes a text (web article, research paper, etc.) and provides options for the type of concept map to produce (fully written out as supplementary material or with deliberate blanks and gaps as exercises). The skill includes instructions to modify the concept map generated based on the discipline the provided text belongs to. Generates a single standalone HTML file that is portable and can be shared with students.

## Inputs Required
One or more texts that the concept map should be generated from.

## Notes and Limitations
* Due to the complexity of the task and the length of the skill instructions, use more capable models (e.g., Claude Opus) for better results. Anthropic/Claude models seem to work more consistently.
* Generating the concept map can be token-intensive. Check your remaining available usage before proceeding.
* Correcting mistakes in the output one at a time can be costly in terms of tokens, as the code for the concept map is regenerated entirely each time. Instead, consolidate all fixes and changes needed within the same prompt as far as possible.

## Attribution
* Author:
Isaac Koh, Centre for Teaching Excellence, SMU
