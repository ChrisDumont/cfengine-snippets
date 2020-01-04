**Raison d'etre**
I needed to edit an XML file and while there is documentation in the cfengine.com reference I couldn't find any practical examples on the web. I hope my example script may help other cfengine users.

**Introduction**
Cfengine is capable of editing an xml file and producing valid machine-readable output. But the XML file could very end up with very long lines that are hard for a human to read. 'xmllint' is capable of formatting it into a human-readable file.  But do I let cfengine edit the file and then manually run xmllint over it? No, of course not.

Normal cfengine practice for any task is to do everything in a script. In editXML.cf, I create our target XML file if necessary, edit the content, and then, if change has occurred, call xmllint. And, as usual, the script must be convergent.

The interaction between an 'edit_xml/insert_tree' promise and using xmllint can easily be non-convergent. The result is an output file that grows each time standalone.cf is run. That's generally considered bad.

The 'first' bundle co-exists nicely with xmllint but putting everything on one line could make the cfengine script hard to read.

In the 'second' bundle I show that having no leading spaces in the 'insert_tree' quote lines lets the pretty formatting by 'xmllint' not interfere with convergence. And this bundle is more readable.

I've deactivated "bundle edit_xml badEdit" by being commenting it out in the 'alone' bundle; uncomment it and run 'standalone.cf' multiple times if you want to see non-convergence in action. In 'badEdit' I naively pretty format our xml. 'xmllint' will slightly change 'some.xml' and the next run of this script is unable to recognize it and re-inserts it. The result: 'some.xml' will grow with each run of this bundle.

**Notes**
Intended audience: those already comfortable with cfengine.

I use Debian so some paths and conventions may be specific to that distro. BTW, 'xmllint' comes from the package 'libxml2-utils'.

Options to run the script:
chmod 755 ./standalone.cf
cf-promises -f ./standalone.cf                         # Check for coding and syntax errors.
./standalone.cf 										                   # Runs quietly but outputs runtime errors.
cf-agent -K -I -f ./standalone.cf                      # My favourite way to run
cf-agent -K -v -f ./standalone.cf | tee /tmp/cf3output # Runs verbosely.

**Credits**
The 'initialize' and 'first' bundles are straight out of the cfengine.com reference. 

