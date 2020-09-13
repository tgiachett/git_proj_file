# git_proj_file
testing out versioning of adobe premiere proj file

Main Resource: https://superuser.com/questions/1437774/how-can-i-setup-adobe-premiere-with-git

Yes, you can use a "textconv" filter, which will be used by git to convert files into a diff-able format when displaying. See:

https://git.wiki.kernel.org/index.php/Textconv
https://git-scm.com/docs/gitattributes
In short, you first need to define the filter via git config (always untracked):

git config --global diff.gzipped.textconv "gunzip -c"

[diff "gzipped"]
    textconv = gunzip -c
Then assign it to .prproj files via .gitattributes (tracked) or .git/info/attributes (untracked):

*.prproj diff textconv=gzipped
(The diff attribute is needed to enable diffing altogether, i.e. tell git that this file should be treated as text and not binary.)

You can also use this method to make more advanced converters, e.g. additionally canonicalize and prettify the XML (if the program writes everything as a single long line).

However, textconv filters are display-only. Storing a compressed file in Git still has the disadvantage that there is no similarity between versions, so if the file is also large, it'll cause the repository to grow accordingly with each version. (Compare this to text files, which are usually very similar between versions, therefore even a long commit history can be very efficiently delta-compressed.)

You might instead want to use "smudge/clean" filters, which work in a similar way but their results are permanent – that is, git stores a 'cleaned' version internally, and transforms it back (smudges) on checkout. The config would look like:

[filter "gzipped"]
    clean = gunzip -c
    smudge = gzip -c
And the attributes file would look like:

*.prproj filter=gzipped
This lets you take advantage of Git's delta compression, although also has the disadvantage that every clone must have the filter configured in their git config (it can't be automatically distributed). If someone clones the repo without having the filters yet, they will need to git checkout --renormalize.

Again, you can expand the 'clean' filter to also prettify the XML (xml_pp) and make it diff in a visually nicer way.
