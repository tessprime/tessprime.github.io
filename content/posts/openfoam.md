---
title: "Openfoam"
date: 2023-05-01T16:48:00-07:00
---
# Goals
Goals for openfoam for this week:

- [x] Get openfoam running on local machine
- [ ] visualize output
- [ ] get basic fluid flow simulation working
- [ ] get basic fluid flow simulation with rotary thing
- [ ] get basic fluid flow simulation with free floating rigid object

# Progress
## Getting openfoam up and running
Okay so apparently you need to do some gpg magic

```
sudo sh -c "wget -O - https://dl.openfoam.org/gpg.key > /etc/apt/trusted.gpg.d/openfoam.asc"
sudo add-apt-repository http://dl.openfoam.org/ubuntu
```

And then you maybe need to install `openfoam` and `openfoam10`. I found the latter actually seems to install
stuff, unsure if the former is useful.

Once you do that, the recommendation is to copy one of 
the tutorial directories in openfoam and run the `Runall`
script to build stuff.

The documentation is... less than stellar.

## Visualize Output

Got it about as far as figuring out how to view the mesh and initial conditions I think. There's a youtube
video (blech) describing how to do an animation.
