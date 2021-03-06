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

5. [Install DOCK 6.7](http://jovingelabsoftware.github.io/blog/2016/09/22/installing-dock-6-7-with-parallel-support/).  The following assumes that the dock applications are in your path.  To avoid future issues with installing new versions, I like to create symlinks to the executables (in particular, sphgen, showshphere, showbox, and dock6) somewhere on your PATH (e.g. /usr/local/bin or, without root access, ~/local/bin, assuming you have created those folders in your home directory and added ~/local/bin to your PATH.)


Phew!  Now you are ready to rock.  I mean dock.  The docking procedure has the following steps:

1. Clean up your protein structure to remove ligands, solvents and select a single member of oligomers.
2. Run chimera's dockprep to add hydrogens if needed, fix incomplete residues, etc.
3. Generate a sphere file to identify the binding pocket
4. Generate a bounding box to limit the search to the region of the largest sphere cluster
5. Generate the search grid
6. Dock.

### Clean up protein structure

First let's prepare our protein (the "receptor").  Many of the structures available from [PDB](http://www.rcsb.org/) are dimers or oligomers and also may contain solvent molecules and even ligands.  So we need to clean them up and extract a single chain.  (You may actually not want a single chain.  Perhaps you are interested in docking between subunits, for example.  In that case you would need to adapt the strategy below.  Of course, the larger the protein assembly you are working with, the longer some of the downstream steps will take computationally).

This script (which requires the little jlab library below) will extract a single chain (indicated by chain letter) and discard everything else including solvent, ligands, etc.

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

The code including our jlab library is [available on github](https://github.com/JovingeLabSoftware/synergy), but here is the guts of it:

**`jlpdb.py`**
```python
import os
from Bio import PDB
import re


"""
Adapted from David Cain's answer here:
http://stackoverflow.com/questions/11685716/how-to-extract-chains-from-a-pdb-file
"""

class Splitter:
    def __init__(self, out_dir=None):
        self.parser = PDB.PDBParser()
        self.writer = PDB.PDBIO()
        if out_dir is None:
            out_dir = os.getcwd()
        self.out_dir = out_dir

    def make_pdb(self, pdb_path, chain_letter, overwrite=False, struct=None):
        chain_letter = chain_letter.upper()
        base = pdb_path.replace("\\\\", "/").split("/").pop()

        # Input/output files
        (pdb_dir, pdb_fn) = os.path.split(pdb_path)
        pdb_id = pdb_fn[3:7]
        out_name = base.replace(".pdb", "_" + chain_letter + ".pdb")
        out_path = os.path.join(self.out_dir, out_name)

        # Skip PDB generation if the file already exists
        if (not overwrite) and (os.path.isfile(out_path)):
            print("Chain %s of '%s' already extracted to '%s'." %
                  (", ".join(chain_letter), pdb_id, out_name))
            return out_path

        # Get structure, write new file with only given chains
        if struct is None:
            struct = self.parser.get_structure(pdb_id, pdb_path)
        self.writer.set_structure(struct)
        self.writer.save(out_path, select=_selectChains(chain_letter))

        return out_path


class _selectChains(PDB.Select):
    def __init__(self, chain_letter):
        self.chain_letter = chain_letter

    def accept_chain(self, chain):
        return (chain.get_id() in self.chain_letter)

    def accept_residue(self, residue):
        # reject all non protein residues
        return  residue.get_id()[0] == ' '

```

### Dockprep

The resulting protein structure is probably not guaranteed to be ready for docking.  I am not sure the structures from PDB always have all the hydrogens added, and there may be incomplete side chains.  (Maybe these are not valid concerns, but for now let's play it safe).  Chimera includes a nice suite of tools called DockPrep, and you can run them from their GUI.  But we are aiming for a headless/unsupervised pipeline here.  Fortunately, you can also script most (all?) of Chimera's functionality with python.  It actually is very simple.  The following preps the protein model, and then also calculates the accessible surface (which if I understand correctly is a smoothed van der Waals surface which excludes tight crevices).

**`dockprep.py`**
```python
import chimera
import sys
import re
from DockPrep import prep
from DockPrep import AddH
from WriteMol2 import writeMol2

# ref: http://mailman.docking.org/pipermail/dock-fans/2007-May/001043.html
models = chimera.openModels.list(modelTypes=[chimera.Molecule])
prep(models, addHFunc=AddH.hbondAddHydrogens)

root = models[0].name.replace(".pdb", "")
if len(sys.argv) == 2:
     d = re.sub("/$", "", sys.argv[1]) + "/"
else:
     d = ""

writeMol2(models, d + root + "_prepped.mol2")

# ref: http://plato.cgl.ucsf.edu/pipermail/chimera-users/2011-March/006134.html
chimera.runCommand("surf")
surf = chimera.openModels.list(modelTypes=[chimera.MSMSModel])[0]
from WriteDMS import writeDMS
writeDMS(surf, d + root + ".dms")
```

### Generate spheres

The next step is to generate the spheres from the surface to identify docking location.  The `sphgen` application (part of DOCK) requires [a config file](http://dock.compbio.ucsf.edu/DOCK_6/tutorials/sphere_generation/generating_spheres.htm), INSPH, to run in non-interactive mode (this file must be called INSPH, and must be in your current working directory.  There is not option to use a different filename at this time).  The name of the `dms` file (`out/mol.dms`) will need to be adjusted to match the file output by `dockprep.py` above.

**`INSPH`**
```
out/mol.dms
R
X
0.0
5.0
1.4

```

Then the spheres can be generated with `sphgen`.  If you want to visualize the spheres, you can convert the sphere file to pdb with the `showsphere` utility that comes with Dock.  Again, to run in non-interactive mode an input file is required:

**`sphgen_cluster.in`**
```
out/4Z3K_A.sph
1
N
selected_cluster.pdb
```

Next, we want to trim our sphere file so it only includes the largest cluster.  This will results in a smaller search box and faster docking.  You don't have to do this if you are concerned your actual binding site is in some other cluster but it seems like A Good Idea (tm).  We can use sed to accomplish this, along these lines:

```
sed -n '/cluster     2/!p;//q' out/4Z3K_A.sph > temp.sph
mv temp.sh out/4Z3K_A.sph
```

### Generate bounding box

To generate the boundnig box surrounding our spheres/binding pocket, we use the `showbox` utility that comes with dock.  As with the other utilities, we need an input file to run non-interactively.  These are, in order, the answers to the prompts that [showbox would ask](http://dock.compbio.ucsf.edu/DOCK_6/tutorials/grid_generation/generating_grid.html).

**`box.in`**
```
Y
5
./out/4Z3K_A.sph
1
./out/rec_box.pdb
```

If you open up the resulting prepped `.mol2` file and the `.pdb` files created by `showcluster` and `showbox` you can see how things are set up for docking (you will need the change the atom display mode to spheres our your spheres will display as little dots).

![dockprep.png]({{site.baseurl}}/media/dockprep.png)




