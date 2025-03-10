---
title: Goose on weekend side projects
date: 2025-03-10
img:
---
I spent some of my limited spare time between childcare duties working on a project: creating llvm-binary-releases using Goose. The goal was straightforward: take an LLVM release package, extract the specific clang tool binary I needed, and produce a new platform/OS-specific release. I planned to set up a hermit package for clang-format because finding pre-built clang binaries is surprisingly difficult. While native package managers are available, they tend to complicate dependency management in build systems, so I decided to solve the problem myself.

Starting with Goose was simple. I asked it to generate a script to download a tarball from a given URL and extract the required binary. It produced about a 100-line bash script that worked well for a task confined to a single source file. However, when I moved on to more complex tasks—such as incorporating bash functions—issues began to appear. Instead of returning proper error codes, Goose used echo to return strings, which caused problems when other parts of the program attempted to use these values. It seemed that Goose had difficulty adhering to the API contracts it had designed. I wondered how other AI tools would handle this situation, but I managed to resolve the problem by asking Goose to rewrite the code in Python. The Python version maintained the API contracts properly.

Next, I requested that Goose create a GitHub workflow to handle the output from the Python script. This task proved challenging. After several rounds of error reports and fixes, Goose made some adjustments, but the changes were sometimes suboptimal, and it often failed to interpret the error messages correctly. In the end, I fixed the issue myself—it turned out to be a one-line problem after I reviewed the documentation.

Overall, the project took about three hours over the weekend, even with interruptions from childcare. Although the final product isn’t perfect, I’m satisfied with the result considering the time and effort involved. In previous experiences, similar projects might have required 3-4 hours of focused work, and I might have abandoned the project when faced with obstacles. For me, using LLMs has proven valuable in jumpstarting and completing these side projects, automating tedious tasks and even producing solid documentation.

Now, I’ll copy this draft into another LLM for a final polish.

*PS: I copied this draft into the chatgpt for polishing, and did NOT add the "another LLM" part, just the line of copying this into LLM. And chatgpt recognized Goose as a different LLM and differentiated itself. Anyways just thought it was an interesting detail.*
