---
layout: post
category: blog
date: '2016-10-05 14:41 -0400'
author: JLab Skunkworks
published: false
title: 'What''s up, DOCK?'
tags: ''
---
## A small molecule docking pipeline

Here are the steps we followed to setup our docking pipeline, with what are hopefully useful pointers along the way.

1. Install Python (e.g. `apt-get install python2.7`)
2. Install numpy (e.g. `apt-get install python-numpy`)
3. [Install biopython](http://biopython.org/wiki/Download)

 Note that if you do not have root access, you will want to build these from source, following the general  approach for configuring for a local install directory I described earlier [in another context](http://jovingelabsoftware.github.io/blog/2016/02/15/make-r-locally-with-cairo/).  For installing python packages using the python package manager (`pip`) [see this post](http://stackoverflow.com/questions/2915471/install-a-python-package-into-a-different-directory-using-pip).

4. Install chimera (headless version, which is listed under ["daily builds"](https://www.cgl.ucsf.edu/chimera/download.html#daily), or try the direct link [here](https://www.cgl.ucsf.edu/chimera/cgi-bin/secure/chimera-get.py?file=alpha/chimera-alpha-linux_x86_64_osmesa.bin)).  This is a user friendly installer that will let you specify the installation directory AND create symlinks somewhere in your path.  So it works great whether or not you have root access on your machine.

5. [Install DOCK 6.7](http://jovingelabsoftware.github.io/blog/2016/09/22/installing-dock-6-7-with-parallel-support/)


Phew!  Now you are ready to rock.  I mean dock.  First let's prepare our protein (the "receptor").  Many of the structures available from [PDB](http://www.rcsb.org/) are dimers or oligomers and also may contain solvent molecules and even ligands.  So we need to clean them up and extract a single chain.  (You may actually not want a single chain.  Perhaps you are interested in docking between subunits, for example.  In that case you would need to adapt the strategy below.  Of course, the larger the protein assembly you are working with, the longer some of the downstream steps will take computationally).

**`pdbsplit.py`**
```python
import sys
sys.path.insert(0, 'lib/python')

from jlab import jlpdb

if __name__ == "__main__":
    """ Parses PDB files desired chain, and creates new PDB structures,
        stripping all non-protein residues (inculding solvents and ligands)
    """
    import sys

    if not len(sys.argv) = 3:
        print "Usage: $ python %s source chain [outdir]" % __file__
        sys.exit()

    if len(sys.argv) == 4:
        splitter = jlpdb.Splitter(sys.argv[3])
    else:
        splitter = jlpdb.Splitter()

    splitter.make_pdb(sys.argv[1], sys.argv[2])

```


