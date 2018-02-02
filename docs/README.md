# Graphyt: A 3D MPM code for continuum simulations

## Highlights
- __Python__ Interface for flexible and fast input scripting
- __Pyck__ Support for pre-preocessing
- __3D__ Material Point Method (**MPM**) Simulation
- __Explicit/Leap Frog__ time integration
- __OpenMP__ Parallel Processing
- __VTP/VTI/CSV__ output formats (some require additional libraries - see below), can be viewed in ParaView

## Current Physics Supported:
- Solid Mechanics: small strain theory
    - Plasticity: Drucker-Prager, Perfect
    - Damage: Grady-Kipp
- Fluid Mechanics: Newtonian fluids
- Fluid-Solid Interaction
- Multi-body Contact
- Rigid Bodies (moving & fixed)
- Thermal conductivity


## Building/Running the Code

#### Obtaining the code
The code can be obtained by either:
- __Downloading it directly__ from Geogit as a compressed archive (e.g. zip file).
- __Cloning the git repository__: using the command 
-  git clone git@geogit.mit.edu:sjr/Graphyt.git

#### Compiling the code
- Create a directory in the Graphyt root folder (eg. build), cd build and type "cmake ../"
- Type make

#### Running the code
- Make an output directory "output" in the same folder as the python scripts to run
- Create a python script of the model
- Type "python nameOfScript.py" in the terminal

## Code Features
 - __Interpolation__ is based on simple GIMP implementation 
 - __Boundary Conditions__ can be applied to particles (vel, stress, force(accel)) and nodes (vel, force). Currently only one BC type per particle/node
 - __Domain__ is periodic by default, for finite simulations, nodal BCs can be applied within 2 cells of the domain extent defined in the scripts


 ## Creating Visualisations in Houdini
 - Output data using VTI file format
 - Open files in Paraview:
                        - Create a 'threshold'
                        - Select filter: 'Cell to Point Data'
                        - Create a contour
                        - Sava Data as ".stl"
                        - Import stl's into Houdini


# Graphyt API Documentation
## MPM Simulation Engine
# Python Modules
Python modules are required for a graphyt script. Aside from the graphyt module, for more complex geometries, pyck (detailed below) may be used, for visualization, the VTKWriter module (also detailed below) should be included. 
```python
import graphyt
import pyck  # if using pyck for geometry
import VTKWriter # for writing output files
```

# Domain and Resolution
To initialize the background grid within which MPM simulations occur, the domain and resolution need to be defined:
```
L        = [Lx,Ly,Lz] # Lx,y,z are the lengths of the grid
                 in each dimension in meters
cellsize = dx # This is the distance between the nodes
                (only uniform resolution supported)
psep     = dx / 2 # This is the distance between particles
                    (optimal value is dx/2)
```
# Material Points and Nodes
The nodes and material points' properties are controlled and contained in the graphyt.Nodes and graphyt.MaterialPoints python object. They require some input values during creation as shown below.
```python
nodes     = graphyt.Nodes([Lx,Ly,Lz], cellsize)
matpoints = graphyt.MaterialPoints(nodes)
```
These objects are used for the majority of the rest of the simulation process. 
# Defining Materials

A number of material models are available in graphyt. To define a material, the format follows the same pattern:<br/>
```python
# Create the material object and set the material-type
    material_1 = graphyt.MaterialModels()
    material_1.setModel(1) 
# 1 - Elastic
# 2 - Newtonian Fluid
# 3 - Elastic + Perfect Plastic
# 4 - Elastic + Drucker Prager Plasticity
```
## Material Models 
___
```python
# Elastic
    material_1.setBulkMod(val)
    material_1.setShearMod(val)
```
```python
# Newtonian Fluid
    material_1.setBulkMod(val)
    material_1.setViscosity(val)
```
```python
# Elastic + Perfect Plastic
    material_1.setBulkMod(val)
    material_1.setShearMod(val)
    material_1.setYieldStress(val)
```
```python
# Elastic + Drucker Prager
    material_1.setBulkMod(val)
    material_1.setShearMod(val)
    material_1.setCohesion(val)
    material_1.setCriticalStrain(val)
    material_1.setFrictionAngle(val)
    material_1.setDilatancyAngle(val)
```

## Damage Models
Damage models can be added to both elastic and elastic-plastic solids.
___
```python
# Grady-Kipp
    material_1.setIsDamage(boolean)
    material_1.setDamageModel(val)
    material_1.setCrackSpeed(val)
    material_1.setDamageM(val)
    material_1.setDamageK(val)
```
___
```python
# Set Material Density
    material_1.setDensity(val)
# Add the material to a materials array, more than one material can be defined
    materials = [material_1]
```
# Geometry
#PYCK
*required
    objectID, materialID
```python

```
# non-PYCK (matpoints.add())
*required
    objectID, materialID
```python

```

# Boundary/Initial Conditions
## Nodes
*required
    `node.setBC(nodeID,graphyt.BCTypes.grid_type,[vals])`
```python

```
## Material Points
```python

```    
# Simulation Parameters
*required
    `is3D,tmax, graphyt.Parameters(max,is3D)`
```python

```

# Set Up I/O
*required
    set arrays, `VTPWriter.VTPWriter(L,cellsize,numParticles)`
```python

```

# Solver 
*required
    `graphyt.Solver(nodes, matpoints, parameters, materials)
    solver.iterate_CPU(t, step,parameters)`
```python

```
# Example: 2D Dam Break
```python
# Import the necessary modules
import graphyt
import pyck
import VTPWriter
import math
## GEOMETRY ##

# Set the resolution and the pack
cellsize = 0.02
psep = cellsize/2.0
# Background Grid
L = [1.0,1.0,0.0]
nodes = graphyt.Nodes(L, cellsize)
print("+++++ Grid INFO+++++ No. Nodes:", nodes.numNodes)
# Material Points
matpoints = graphyt.MaterialPoints(nodes)
matpoints.psep = psep
# PYCK
# create materials
water = graphyt.MaterialModels()
water.setBulkMod(1e6)
water.setShearMod(0.0)
water.setDensity(1000)
water.setModel(2)
materials = [water]
# create packers and pack
cubic = pyck.CubicPacker(L,psep,[0,0,0])
pack = pyck.StructuredPack(cubic)
# create shapes
liquidLowerLeft = [2.75*cellsize,2.75*cellsize,0.0]
liquidUpperRight = [0.5*L[0],0.75*L[1]-2*cellsize,0.0]
liquid = pyck.Cuboid(1,liquidLowerLeft,liquidUpperRight)

# Add shapes to pack
pack.AddShape(liquid)
pack.Process()
model = pyck.Model(pack)
# Set the objectID's for all objects
objID = model.CreateIntField("objectID",1)
model.SetIntField(objID,1,0) 
# Set the material ID for all objects
matID = model.CreateIntField("material",1)
model.SetIntField(matID,1,0) 
# Add the geometry from pyck to the matpoints
matIDfield = model.GetIntField(matID)
objIDfield = model.GetIntField(objID)
numParticles = pack.GetNumParticles()
pyckPos = pack.GetPositions()
matpoints.addPyckGeom(pyckPos,numParticles)
matpoints.addPyckIntField(matIDfield)
matpoints.addPyckIntField(objIDfield)
print("+++++ MAT POINT INFO+++++ No. Particles:", matpoints.numParticles)

# /////////-----------BOUNDARY CONDITIONS--------------///////
# //// MATERIAL POINT BOUNDARY CONDITIONS /////
# //// NODAL BOUNDARY CONDITIONS /////
#// Wall boundaries for Grid Nodes
wallthickness = 2.0*nodes.cellsize
for  n in range(nodes.numNodes):
    posX = nodes.getPosX(n)
    posY = nodes.getPosY(n)
    posZ = nodes.getPosZ(n)
    if (((posX>=L[0]-wallthickness) or (posX <= wallthickness)) or ((posY>=L[1]-wallthickness) or (posY <= wallthickness))):
        nodes.setBC(n,graphyt.BCTypes.grid_vx_vy,[0,0])
print("+++++ MODEL  INFORMATION+++++ Boundary Conditions Applied")

# /////////// Parameters Set /////////////// #
tmax = 5   # //(time in seconds)
is3D = False
parameters = graphyt.Parameters(tmax,is3D)
parameters.setGravity(0, -10, 0)
parameters.setCFL(0.0001)
parameters.setDampingCoef(0.5)
print("+++++ MODEL  INFORMATION+++++ Parameters Set")

# // //+++++++++++++++++++++++++++++++++++++++++++++++//
# // //+++++++++++++ MODEL RUN +++++++++++++++++++++++//
# // //+++++++++++++++++++++++++++++++++++++++++++++++//
#Now Mat points and nodes are set and ready to go, create a solver to run an MPM analysis on them
print("+++++ SOLVER  INFORMATION+++++ Starting Solver")
solver = graphyt.Solver(nodes, matpoints, parameters, materials)

matpntPos = matpoints.getPosArr()
matpntMat = matpoints.getMatArr()
matpntMass = matpoints.getMassArr()
matpntVel = matpoints.getVelArr()
matpntStress = matpoints.getStressArr()
Le = [1+int(L[0]/cellsize), 1+int(L[1]/cellsize), 1+int(L[2]/cellsize)]
vtp = VTPWriter.VTPWriter(Le,cellsize,numParticles)
vtp.AddArray("Position", matpntPos,3, 2)
vtp.AddArray("Material", matpntMat,1, 2)
vtp.AddArray("Mass", matpntMass,1, 2)
vtp.AddArray("Velocity",matpntVel,3, 2)
vtp.AddArray("Stress",matpntStress,6, 2)


# Iterate over nsteps number of timesteps
for step in range(int(solver.tmax/solver.dt)):  # ; t<solver.tmax; t+=solver.dt):
    t = solver.dt * step
    solver.iterate_CPU(t, step, parameters)
    vtpfname='output/dambreak2D'+str(step)+'.vtp'
    if step % 50000 == 0:
        vtp.Write(vtpfname)

print("Total Simulation Duration (s)")
```

