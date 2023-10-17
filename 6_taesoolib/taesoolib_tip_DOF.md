
날짜 : 2023-10-12
태그 :   #taesoolib 
출처 :   
저자 :   
url :   
인용 :   
위치 :  
연결문서 :   [[taesooLib_documentation_main]]


# 잡설

pose를 결정할 때 우리는 SetDOF 에 vectorn 을 넣어서 설정을 하는데 그 벡터의 크기를 궁금해 할 수 있다.

다음을 생각하면 된다.

벡터의 크기 = 조절할 수 있는 관절 수 + 이동할 수 있는 벡터 수

여기서 회전 할 수 있는 관절 수 는 wrl의 JointAxis 의 개수의 총 합을 보면 되고, 이동 할 수 있는 본의 수는 'jointType' 이 "free" 인 본의 개수를 보면 된다.

# 튜토리얼 코드

```lua
require("config")

require("common")

  

package.projectPath='../Samples/QP_controller/'

package.path=package.path..";../Samples/QP_controller/lua/?.lua" --;"..package.path

package.path=package.path..";../Samples/classification/lua/?.lua" --;"..package.path

package.path=package.path..";../Resource/classification/lua/?.lua" --;"..package.path

require("gym_qp/PDservo")

require('gym_qp/RagdollSimPD')

require("tl")

require("gym_qp/module/RetargetConfigPreset")

timestep=1/480 -- multiples of 30

--timestep=1/120 -- multiples of 30

  

rendering_step=1/30

  

--model=model_files.default model.initialHeight=0.0

model=model_files.gymnist model.initialHeight=0.07

model.frame_rate=30 -- motion data framerate (important because this is used when calculating derivatives)

model.k_p_PD=500 -- default: 500, 5

model.k_d_PD=10

model.start=0

model.muscleActiveness=1.0

--model.file_name="gym_gang/Resource_RL/kickball/hyunwoo_lowdof_T.wrl"

--model.file_name="../Resource/motion/locomotion_hyunwoo/hyunwoo_lowdof_T3.wrl" --

model.file_name="../Resource/motion/locomotion_hyunwoo/hyunwoo_lowdof_T_boxfoot.wrl" --

model.mot_file="gym_gang/Resource_RL/kickball/kickball_hyunwoo.dof" -- from commit aa695401d

model.fixDOF=fixDOF_kickball

  

float_options={}

float_options.impulseMagnitude={ val=100, min=10, max=1500}

float_options.impulseDuration={ val=0.2, min=0, max=1}

  

drawMode=true

nanchecker=false

target=vectorn()

  

randomRestart=false

randomRestartInfo={prevStart=model.start, freq=3, limitLength=90}

-- freq will be adjusted to 1 when testing (make cpu_qp2).

  

randomInitial=false -- do not change here.

random_action_environment=false

random_action_max=1

  

hasGUI=false

  

-- RL environment functions:

-- reset, init_env, step

  

function generateRandomNumbers()

pseudoRandom=vectorn(1000)

math.randomseed(0)

for i=0, 1000-1 do

pseudoRandom:set(i, math.random())

end

end

  

function ctor()

generateRandomNumbers()

if isMainloopInLua then

hasGUI=true

end

mEventReceiver=EVR()

this:create("Button", "viewPoint", "viewPoint")

this:create("Check_Button", "Simulation", "Simulation", 0, 2,0)

this:widget(0):checkButtonValue(0) -- 1 for imediate start

this:widget(0):buttonShortcut("FL_CTRL+s")

this:create("Button", "capture", "capture")

this:create("Button", "viewpoint", "viewpoint")

this:create("Button", "start randomRestart", "start randomRestart",0,3,0)

this:create("Button", "start randomInitial", "start randomInitial")

this:create("Button", "start zero action", "start zero action")

this:create("Button", "show reference motion", "show reference motion")

this:create("Button", "show original motion", "show original motion")

  

for k, v in pairs(float_options) do

this:create("Value_Slider" , k, k,1)

this:widget(0):sliderRange(v.min, v.max)

this:widget(0):sliderValue(v.val)

end

  

this:create("Button", "push", "push",1);

  

this:updateLayout()

collisionTestOnlyAnkle=false

debugContactParam={10, 0, 0.01, 0, 0}-- size, tx, ty, tz, tfront

end

  

if EventReceiver then

EVR=LUAclass(EventReceiver)

function EVR:__init(graph)

self.cameraInfo={}

end

end

  

function onCallback(w, userData)

if w:id()=='viewPoint' then

print('view pos',RE.viewpoint().vpos)

print('view at',RE.viewpoint().vat)

elseif w:id()=='push' then

local simulationFrameRate=1/timestep

  

simulator.impulse=float_options.impulseDuration.val*simulationFrameRate

simulator.impulseDir=vector3(1,0,0)*float_options.impulseMagnitude.val

simulator.impulseGizmo=simulator.objectList:registerEntity("arrow2", "arrow2.mesh")

simulator.impulseGizmo:setScale(2,2,2)

RE.output("impulse", tostring(simulator.impulse))

elseif w:id()=="show reference motion" then

if not simulator then

init_env()

end

  

simulator.skin:applyMotionDOF(simulator.motionDOF)

RE.motionPanel():motionWin():addSkin(simulator.skin)

elseif w:id()=="show original motion" then

if not simulator then

init_env()

end

  

simulator.skin:applyMotionDOF(simulator.motionDOF_original)

RE.motionPanel():motionWin():addSkin(simulator.skin)

  

elseif w:id()=='start randomRestart' then

generateRandomNumbers()

randomRestart=true

randomRestartInfo.freq=1

elseif w:id()=="start randomInitial" then

randomInitial=true

elseif w:id()=='capture' then

RE.renderer():screenshot(true)

elseif w:id()=='viewpoint' then

print(RE.viewpoint().vpos)

print(RE.viewpoint().vat)

elseif w:id()=="start zero action" then

random_action_environment=true

random_action_max=0

hasGUI=true

g_stepCount=0

init_env()

  

if true then

-- print debug information

___reset_old=reset

reset=function()

local return_initial_state=___reset_old()

-- if you want to test the step function outside of render loop.

print ('reset:', return_initial_state)

local dims=get_dim()

local adim=dims(1)

local action=CT.zeros(adim)

out=step(model.start, action)

  

g_stepCount=g_stepCount+1

print('step1:', out)

return_initial_state=out

return return_initial_state

end

end

reset()

else

for k, v in pairs(float_options) do

if w:id()==k then

float_options[k].val=w:sliderValue()

break

end

end

end

end

  

function init_env()

print('lua initenv')

init_env_part1(true)

end

  

function get_dim()

return CT.vec(47,25)

end

  
  

function frameMove(fElapsedTime)

if isMainloopInLua then

python.F('test_gym_qp', 'envStep')

elseif random_action_environment then

-- python에서 초출 되는 순서로 호출함

  

local dims=get_dim()

local adim=dims(1)

local action=(CT.rand(adim)-CT.ones(adim)*0.5)*(random_action_max*2)

  

local step_state, episode_done, step_reward=step(g_stepCount, action)

  
  

g_stepCount=g_stepCount+1

if episode_done==1 then

g_stepCount=0

reset()

end

end

  

end

  

function EVR:onFrameChanged(win, iframe)

end

  

function step(_python_iframe_unused, action)

local iframe=simulator.RLenv.iframe

--assert(randomRestart or (_python_iframe_unused==iframe))

  

if action:isnan() then

print('action nan??? .. using random action')

for i=0, action:size()-1 do

action:set(i, math.random()-0.5)

end

end

local targetAction=vectorn()

targetAction=actionScaling(action)

  

local res, step_reward, step_state, done_

res, step_reward, step_state, done_= pcall(RagdollSim.frameMove, simulator, iframe, targetAction)

if type(step_reward)=='string' then

print('error!', step_reward) -- error!

if select(1, string.find(step_reward, 'projectAngle')) then

return CT.zeros(get_dim()(0)),1 , 0

else

assert(false)

end

elseif step_state:isnan() then

print('nan state', action, step_state)

step_state:setAllValue(0)

step_reward=0

end

local mocapstep_per_controlstep=model.frame_rate*rendering_step

simulator.RLenv.iframe=iframe+mocapstep_per_controlstep

simulator.globalTime=simulator.globalTime+rendering_step

  

if isMainloopInLua and model.loopMotion then

local maxHistory=model.frame_rate/2

if iframe+mocapstep_per_controlstep-maxHistory>simulator.motionDOF_original:numFrames()-1 then

-- transition

simulator.RLenv.iframe=simulator.RLenv.iframe-

(simulator.motionDOF_original:numFrames()-1)

-- update replayBuffer

local delta=iframe-simulator.RLenv.iframe

local replayBuffer=simulator.replayBuffer

--RE.output2('updaterepl', iframe, simulator.RLenv.iframe, 0, maxHistory)

for i=0, maxHistory+1 do

replayBuffer:row(i):assign(replayBuffer:row(i+delta))

end

end

end

--

-- ctrl+alt+o to see outputs

RE.output2('RL step', simulator.RLenv.iframe,done_, step_reward, step_state, action)

  

if done_ then

return step_state,1 , step_reward

else

return step_state,0 , step_reward

end

end

  

function python_render(renderBool)

RE.renderOneFrame(true)

end

  

function reset()

local res, return_initial_state= pcall(simulator.reset, simulator)

if not res then

print(return_initial_state)

dbg.console()

end

  

return return_initial_state

end

  

function render()

RE.renderOneFrame(true)

RE.renderer():screenshot(true)

end

  

function state_reScaling(step_state)

local rescale_state=vectorn(step_state:size())

for i=0, step_state:size()-1 do

rescale_state:range(i,i+1):assign((step_state:range(i,i+1)/5))

end

return rescale_state

end

  

function actionScaling(actions)

return actions*0.2

end

  

function getDofIndex(skel)

local DoFIndexData={}

local totalDoFNum=0

print("Treeindex","BoneName","NumDof","startDoFIndex")

for j=1, skel:numBone()-1 do

DoFIndexData.BoneName = skel:getBoneByTreeIndex(j)

  

DoFIndexData.NumDoF = skel.dofInfo:numDOF(j)

  

DoFIndexData.startDoFIndex = skel.dofInfo:DOFindex(j,0)

print(j," ",DoFIndexData.BoneName," ",DoFIndexData.NumDoF," ",DoFIndexData.startDoFIndex)

totalDoFNum=totalDoFNum+DoFIndexData.NumDoF

end

  

print("\n total joint DoF Number is ", totalDoFNum)

--return dofIndex

end

  

---------------------------------after this line, simulation class--------------------------------

  
  
  

function RagdollSim:createTargetpose(refPose, action)

local generatedPos

generatedPos=refPose:copy() --> reference

--calc target pose

--maximum

self:addAction(generatedPos, action)

return generatedPos

end

  

function RagdollSim:frameMove(iframe,action)

self.controlforce:zero()

  

--local timer=util.PerfTimer2()

--timer:start()

  

local refpose=self:createRefpose(iframe)

self:setRefTree(iframe)

local targetpose= self:createTargetpose(refpose,action)

  

--os.sleep(1)

--timer:stopMsg('setPoseDOF') -- type ctrl+alt+o to see the computation time

local character_fall=false

local episode_done=false

  

--timer:start()

local target_delta=targetpose-refpose

--local cfL=0

--local cfR=0

--local tiL=self.loader:getTreeIndexByVoca(MotionLoader.LEFTANKLE)

--local tiR=self.loader:getTreeIndexByVoca(MotionLoader.RIGHTANKLE)

local mocapstep_per_controlstep=model.frame_rate*rendering_step

  
  

for iter=1,niter do

local impulse

if self.impulse>0 then

RE.output2("impulse",self.impulse)

local chest=self.loader:VRMLbone(self.loader:getBoneByVoca(MotionLoader.CHEST):treeIndex())

  

local gf=self.impulseDir

local frame=self.simulator:getWorldState(0):globalFrame(chest)

local lf=frame:toLocalDir(gf)

  

local dir=gf:copy()

dir:normalize()

if self.impulseGizmo then

local t=transf()

t:axisToAxis(vector3(0,0,0), vector3(0,1,0), frame.translation*100-10*dir, dir)

RE.output2("gizmo", t.rotation,t.translation)

self.impulseGizmo:transform(t)

dbg.namedDraw('Sphere', t.translation*100, 'impulse',"red", 5)

else

local pos=frame:toGlobalPos(chest:localCOM())

dbg.namedDraw('Arrow',pos*100-50*dir,pos*100,'impulseGizmo')

end

impulse={chest=chest, lf=lf}

self.impulse=self.impulse-1

end

local maxForce=9.8*80

self.pdservo.currFrame=iframe+mocapstep_per_controlstep*((iter-1)/niter)

self.pdservo.currFrame=math.min(self.pdservo.currFrame, self.pdservo.endFrame-1)

  

if not self.pdservo:generateTorque(self.simulator,target_delta ) then

--self.pdservo:rewindTargetmotion(self.simulator)

--self.pdservo:generateTorque(self.simulator, maxForce)

end

  

if self.pdservo.theta:isnan() then

episode_done=true

break

end

  
  

local controlforce=self.pdservo.controlforce

--self.controltorque=self.pdservo:generateWorldTorque(self.simulator)

--worldAngvel = self.pdservo.angvel

--worldOri = self.pdservo.korientation

-- use only PD servo

assert( self.pdservo.stepSimul )

self.pdservo:stepSimul(self.simulator, nil, impulse)

end

  

if self.impulse<=0 and self.impulseGizmo~=nil then

self.objectList:erase("arrow2")

self.impulseGizmo=nil

end

--timer:stopMsg('simulLoop')

--timer:start()

  

if hasGUI then

-- debug draw

self.skin_ref:setPoseDOF(refpose)

self.skin_target:setPoseDOF(targetpose)

local theta=self.simulator:getLastSimulatedPose(0)

self.skin:setPoseDOF(theta)

  

if attachCamera then

if not g_prevRootPos then

g_prevRootPos=vector3(0,0,0)

end

local COM=self.simulator:calculateCOM(0)

mCOMfilter:setCurrPose(CT.vec(COM))

local curPos= mCOMfilter:getFiltered():toVector3(0)*100

curPos.y=0

RE.viewpoint().vpos:assign(RE.viewpoint().vpos+curPos-g_prevRootPos)

RE.viewpoint().vat:assign(RE.viewpoint().vat+curPos-g_prevRootPos)

RE.viewpoint():update()

g_prevRootPos=curPos:copy()

  

-- floor follows the character on integer grids (meters)

--mFloorSkin:setTranslation(math.round(curPos.x/100)*100, 0, math.round(curPos.z/100)*100)

mFloorSkin:setTranslation(curPos.x, 0, curPos.z)

end

  
  

end

if not episode_done then

episode_done,character_fall=self:episodechecker(iframe, self.tree_ref)

end

--timer:stopMsg('episodecheck')

--timer:start()

self:setRefTree(iframe+mocapstep_per_controlstep) -- 시뮬레이션 한 rendering_Step진행했으니 iframe+1과 비교하는게 맞음

local step_reward=self:calcReward(self.simulator,self.tree_ref, iframe+mocapstep_per_controlstep)

local step_state=simulator:getEnvState()

  

return step_reward, step_state, episode_done

end

  

function RagdollSim:episodechecker(iframe, ref_tree)

local episode_done=false;

local finished=false;

  

local sim_pose=self.simulator:getLastSimulatedPose(0)

for i=0, sim_pose:size()-8 do

if math.abs(sim_pose:get(i+7))>3.2 then

--print('episode over : range over')

episode_done=true

end

end

-- episode_done=true

if sim_pose:get(1)<ref_tree:globalFrame(1).translation.y*(model.fallThr or 0.7) then

--print('episode over : fall')

episode_done=true

elseif iframe>=self.motionDOF:rows()-2 then

episode_done=true

finished=true

-- print('episode over : episode over')

end

if (randomRestart or model.loopMotion) and randomRestartInfo.limitLength and not isMainloopInLua then

-- limitlength is used only when training.

  

if self.globalTime*model.frame_rate> randomRestartInfo.limitLength then

episode_done=true

finished=true

end

end

  

return episode_done, finished

end

  

function RagdollSim:reset()

local startFrame=model.start

  

if randomRestart then

local iter_j=randomRestartInfo.iter_j or 0

local count=math.mod(iter_j, randomRestartInfo.freq+1)

if isMainloopInLua then

iter_j=math.round(math.random()*999)

randomRestartInfo.limitLength =nil

end

local limitLength=randomRestartInfo.limitLength or 30

if isMainloopInLua or count==randomRestartInfo.freq then

local nf=self.motionDOF:numFrames()

  

if false and model.loopMotion then

  

startFrame=math.round(

sop.map(pseudoRandom(math.mod(iter_j,1000)), 0, 1, 0, nf-5-limitLength))

  

else

startFrame=math.round(

sop.clampMap(pseudoRandom(math.mod(iter_j,1000)), 0, 1,

model.start-limitLength*0.5, nf-1-limitLength*0.5)

) -- give more probabilty to select frame 0, and nf-1-limitLength

  

if startFrame>nf-1-limitLength then

startFrame=nf-1-limitLength

end

  

if startFrame<model.start then

startFrame=model.start

end

end

  

if startFrame~=randomRestartInfo.prevStart then

randomRestartInfo.prevStart=startFrame

end

else

startFrame=randomRestartInfo.prevStart

end

randomRestartInfo.iter_j=iter_j+1

end

  

self.RLenv={

iframe=startFrame ,

nf=self.motionDOF:numFrames()

}

self.globalTime=0

  

local iframe=self.RLenv.iframe

--simulation = loader

--ref motion = ref

--set all loader to initial pose

self.loader:setPoseDOF(self.motionDOF:row(iframe))

self:setRefTree(iframe)

  

self.ref_prev=self:getJointStateFromRefTree()

  

self.loader_target:setPoseDOF(self.motionDOF:row(iframe))

self.skin:setPoseDOF(self.motionDOF:row(iframe))

self.skin_target:setPoseDOF(self.motionDOF:row(iframe))

self.skin_ref:setPoseDOF(self.motionDOF:row(iframe))

--self.pdservo:resetPDset(self.motionDOF)

local initialState=vectorn()

initialState:assign(self.motionDOF:row(iframe))

  

-- restart from the origin for easier debugging

local deltaRootPos=vector3(initialState(0), 0, initialState(2))

initialState:set(0,0)

initialState:set(2,0)

  

if randomInitial then

local roottf=MotionDOF.rootTransformation(initialState)

roottf.rotation:assign(quater(math.random(),vector3(0,1,0))*roottf.rotation:offsetQ())

initialState:setQuater(3, roottf.rotation)

end

  

if noisyInitial then

local q=initialState:toQuater(3)

q.x=q.x+(math.random()-0.5)*0.04

q.y=q.y+(math.random()-0.5)*0.04

q.z=q.z+(math.random()-0.5)*0.04

q:normalize()

end

  

self.simulator:setLinkData(0, Physics.DynamicsSimulator.JOINT_VALUE, initialState)

  

--self.simulator:getLinkData(0, Physics.DynamicsSimulator.JOINT_VALUE, initialState)

assert( self.DMotionDOF )

local initialVel=self.DMotionDOF:row(iframe):copy()

if noisyInitial then

for i=0,6 do

initialVel:set(i, initialVel(i)+(math.random()-0.5)*0.04)

end

end

--print('initialVel', initialVel)

self.simulator:setLinkData(0, Physics.DynamicsSimulator.JOINT_VELOCITY, initialVel)

self.simulator:initSimulation() -- necessary

self.sim_prev=self:getJointStateFromSimulator()

  
  

self.pdservo:initPDservo(iframe, self.motionDOF:numFrames(),self.motionDOF, self.DMotionDOF)

--self.simulator:getLinkData(0, Physics.DynamicsSimulator.JOINT_VALUE, initialState)

  

-- only root configuration is stored.

self.replayBuffer=self.motionDOF:matView():sub(0,0,0,7):copy()

for i=0, self.replayBuffer:rows()-1 do

self.replayBuffer:row(i):setVec3(0, self.replayBuffer:row(i):toVector3(0)-deltaRootPos)

end

  

local initialState_env=simulator:getEnvState()

  

self.impulse=0

self.externalForce=vector3(0,0,0)

  

return initialState_env

end

  

function RagdollSim:getRefRotY(theta, iframe)

if self.rotYref then

local rootRotY=(theta:toQuater(3)*self.rotYref:quatViewCol(0):sampleRow(iframe)):rotationY()

return rootRotY

else

local rootRotY=theta:toQuater(3):rotationY()

return rootRotY

end

end

function RagdollSim:getRefCoord(theta, iframe)

local rootPos=theta:toVector3(0)

local rootRotY=self:getRefRotY(theta, iframe)

rootPos.y=0

return transf(rootRotY, rootPos)

end

  

-- return a reference coordinate

function RagdollSim:getEnvState()

local theta= self.simulator:getLastSimulatedPose(0)

  

local step_state=vectorn(get_dim()(0))

  

step_state:range(0, theta:size()):assign(theta)

local refCoord=RagdollSim:getRefCoord(theta, self.RLenv.iframe)

local rootRotY=refCoord.rotation

local numFrames=self.motionDOF:numFrames()

-- state has to be coordinate invariant

step_state:set(0, self.RLenv.iframe/self.RLenv.nf) -- phase

step_state:set(2, 0) -- unused

  

-- rootOri=rotY*offsetQ

step_state:setQuater(3, rootRotY:inverse()*step_state:toQuater(3))

  

local step_state_pos=self:extra_state(refCoord)

  

assert(step_state:size()==step_state_pos:size()+theta:size())

step_state:slice(-step_state_pos:size(),0):assign(step_state_pos)

return step_state

end

  

function RagdollSim:createRefpose(iframe)

local out=vectorn()

if iframe>=self.motionDOF:numFrames() then

dbg.console()

end

self.motionDOF:sampleRow(iframe, out) -- iframe: continuous time. can be 5.5 or 7.25 for example

return out

end

function RagdollSim:extra_state(refCoord)

local return_state=vectorn(12)

local loader_com=vector3()

local left_ankle_pos=vector3()

local right_ankle_pos=vector3()

local com_lv=vector3(0)

  

local worldState= self.simulator:getWorldState(0)

loader_com=refCoord:toLocalPos(worldState:globalFrame(1).translation)

  

local upcast=MainLib.VRMLloader.upcast -- bone to VRMLbone

local bL=upcast(mLoader:getBoneByVoca(MotionLoader.LEFTANKLE))

local bR=upcast(mLoader:getBoneByVoca(MotionLoader.RIGHTANKLE))

left_ankle_pos=refCoord:toLocalPos(worldState:globalFrame(bL).translation)

right_ankle_pos=refCoord:toLocalPos(worldState:globalFrame(bR).translation)

com_lv=refCoord:toLocalDir(self.simulator:calculateCOMvel(0))

return_state:range(0,3):assign(loader_com)

return_state:range(3,6):assign(left_ankle_pos)

return_state:range(6,9):assign(right_ankle_pos)

return_state:range(9,12):assign(com_lv)

  

return return_state

end

```