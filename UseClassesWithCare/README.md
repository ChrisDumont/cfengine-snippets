### Raison d'etre
I usually separate my bundles by task. So, on a particular machine I want, say, approx installed so I create a bundle that does everything necessary: install package(s), creating, installing or editing files, and issuing commands. Note the order I stated that in. This is not the order that cfengine does things. The order it does promise types is something like: 'vars', 'classes', 'files', 'packages', and more. 'files' are done before 'packages'. Sometimes that works out; sometimes it doesn't. Repeated runs of cf-agent may eventually sort it out (this is part of an important cfengine concept: convergence) but there are times it doesn't so I need to force things to happen in a different order. I can do this with classes.

### Introduction
Cfengine runs through promise types in three passes: 'vars', 'classes', 'files', 'packages', etc. then 'vars', 'classes', 'files', 'packages', etc. again and again. Judicious use of classes can alter on which pass an individual promise is executed. In this case I want a 'files' promise to run after a 'packages' promise. What really happens is that the 'files' promise is skipped on the first pass, the 'packages' promise installs approx, a class is set, and on the second pass the 'files' promise is executed. There are usually many methods to choose from that will set a class; it's not always obvious which is the right one.

I choose not to use the stock location '/var/cache/approx' since it can become large and therefore put it in a different directory. In this bundle is a 'files' promise will set the owner and group of this directory but must wait until the owner and group exist. I can make the promise not run until the class 'approxExists' is set, but what test do I pick to set that class? I can test with '/usr/bin/id approx', I can test if 'approx' is in /etc/passwd, or I could test if the package approx is installed (the package postinst script creates the user 'approx'). I present four(-and-a-half) variations of the same bundle, the first two fail on the first run, the third succeeds, the fourth can succeed or fail depending upon one detail.

I've also included remove.cf. As you might expect it removes the package and the directory to restore the system to a "green-field" condition. Use it before 'tryXXX.cf' to simulate a first time run on a system.

tryOne.cf demonstrates the failure to get the desired outcome because things don't happen in the right order.

tryTwo.cf has the right idea: only if a class is set is the file ownership promise executed. But tryTwo.cf fails due to the subtlety of caching in cfengine. A 'classes' promise is cached if it can't be done internally, that is, if it calls an external program or shell. The result is that even when some other change would cause the 'classes' promise to have a different outcome on the second (or third) pass the cached original outcome is kept and, thus, a promise that depends on the class being set will not be executed. In tryTwo.cf since the class is not set the change of file ownership is not attempted. I recommend using the verbose method (see below) to run tryTwo.cf so that you can actually see for yourself that cfengine is using a cached result.

tryThree.cf is the same as tryTwo.cf but, instead of '/usr/bin/id' uses 'regline', a cfengine builtin function. The 'classes' promise is therefore evaluated on every pass. In the first pass the class is not set and then the package is installed. On the second pass the class 'approxExists' is set and thus the second pass of the 'files' promise to change the ownership is executed.

tryFour.cf tests if approx is installed to set 'approxExists'. There are a few package tests available; here I've made a wrong choice: 'if_repaired'. On a green-field system tryFour.cf works perfectly. But if, while setting up the system I remembered that I'd be wanting approx I might install it before cfengine is run. Then since it's already installed 'if_repaired' is not true and so 'approxExists' is not set and the 'files' promise isn't run. Edit tryFour.cf and substitute 'if_ok' for 'if_repaired'. Now everything works as desired.

I mentioned near the top that repeated runs of cf-agent might or might not sort everything out. tryOne.sh and tryTwo.cf eventually converge. Run them a second time and you'll get success. tryFour.cf with 'if_repaired' will not succeed no matter how many times it's run.

#### Notes
Intended audience: those already comfortable with cfengine. These scripts, of necessity, run as root and change important things on the system it's run on. The 'approx' package is installed and removed, /etc/passwd and /etc/group are directly edited. If you're not comfortable with that either review these scripts closely or run them on a throwaway box or virtual server.

I use Debian so some paths and conventions may be specific to that distro. Due to a slight incompatibility between Debian Buster and cfengine3-3.12 warnings (W: ... --force-yes ... --allow...) will likely be issued. They are, indeed, warnings and can be safely ignored. UPDATE: My bad. The warning was issued due to my running a mix of cfengine 3.9 (Stretch) and 3.12 (Buster). I had mistakenly used the 3.9 version of the /var/lib/cfengine3/modules/ tree. The 3.12 version has updated the options to apt-get such that there are no more warnings.

Options to run the script:
```
chmod 755 ./tryOne.cf
cf-promises -f ./tryOne.cf                            # Check for coding and syntax errors.
./tryOne.cf                                           # Runs quietly but outputs runtime errors. Beware of 
                                                      # running too soon after previous run due to locking/ifelapsed.
cf-agent -K -I -f ./tryOne.cf                         # My favourite way to run
cf-agent -K -v -f ./tryOne.cf | tee /tmp/cf3Output    # Run verbosely.
```

