
날짜 : 2023-09-26
태그 :   #taesoolib 
출처 :   
저자 :   
url :   
인용 :   
위치 :  
연결문서 :   


# 개요
- OgreMesh_skinning.lua 분석


# Codes


```lua
require("config")

-- 프로젝트 path
package.projectPath='../Samples/classification/'

package.path=package.path..";../Samples/classification/lua/?.lua" --;"..package.path

require("common")

require("module")

  

require("subRoutines/AnimOgreEntity")

  

function ctor()

-- event reciever 시작 

mEventReceiver=EVR()

this:create("Button", "translate skeleton skin", "translate skeleton skin")
this:create("Button", "rotate elbow", "rotate elbow")
  
this:updateLayout()

-- event reciever 끝
  

-- using manual mode.

--local meshFile='X_Bot_fbx.mesh' local entityScale=1

local meshFile='Ch43_nonPBR_fbx.mesh' local entityScale=1

--local meshFile='sth_fbx.mesh' local entityScale=100

  

skinScale=100

local buildEdgeList=true

mOgreEntity=AnimOgreEntity(meshFile, entityScale, skinScale, 1, buildEdgeList)

  

if false then

-- optional debug skin for drawing skeleton.

createDebugSkin()

end

  

if false then

local entity=mOgreEntity

local newPose=Pose()

entity.loader:getPose(newPose)

entity:_setPose(newPose, entity.loader)

end

end

  

function createDebugSkin()

debugSkin= RE.createSkin(mOgreEntity.loader); -- to create character .

debugSkin:scale(skinScale, skinScale, skinScale) -- skeleton

debugSkin:setMaterial('green_transparent')

local pose=Pose()

mOgreEntity.loader:getPose(pose)

debugSkin:_setPose(pose, mOgreEntity.loader)

end

  

function dtor()

mOgreEntity:dtor()

end

  

function handleRendererEvent(ev, button, x, y)

return 0

end

  

function onCallback(w, userData)

  

if w:id()=='translate skeleton skin' then

if not debugSkin then

createDebugSkin()

end

if debugSkin then

debugSkin:setTranslation(100,0,0)

end

elseif w:id()=='rotate elbow' then

local entity=mOgreEntity

local newPose=Pose()

entity.loader:getPose(newPose)

  

local rotJointIndex=entity.loader:getRotJointIndexByName('mixamorig:LeftForeArm')

if rotJointIndex>=0 then

newPose.rotations(rotJointIndex):assign(quater(math.rad(90), vector3(1,0,0)))

if debugSkin then

debugSkin:_setPose(newPose, entity.loader)

end

entity:_setPose(newPose, entity.loader)

else

print('no bone named mixamorig:LeftForeArm')

end

  

end

end

  

if EventReceiver then

EVR=LUAclass(EventReceiver)

function EVR:__init(graph)

self.currFrame=0

self.cameraInfo={}

end

end

  

function EVR:onFrameChanged(win, iframe)

self.currFrame=iframe

  

RE.output("iframe", iframe)

  

end

  

function frameMove(fElapsedTime)

end
````
```

