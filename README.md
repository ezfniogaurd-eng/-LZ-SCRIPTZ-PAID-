------------------------------------------------
-- SERVICES
------------------------------------------------
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local gui = player:WaitForChild("PlayerGui")

------------------------------------------------
-- REMOVE OLD GUI
------------------------------------------------
if gui:FindFirstChild("LZ_UI") then
	gui.LZ_UI:Destroy()
end

------------------------------------------------
-- VARIABLES
------------------------------------------------
local humanoid
local rootPart
local animator
local spawnCFrame
local floatForce

local noLagEnabled = false
local lzBoostEnabled = false
local floatEnabled = false

local BOOST_SPEED = 28.5
local FLOAT_SPEED = 28.8
local FLOAT_HEIGHT_FAR = 5
local FLOAT_HEIGHT_NEAR = 4
local NEAR_DISTANCE = 20
local LIFT_FORCE = 20

------------------------------------------------
-- CHARACTER SETUP
------------------------------------------------
local function setupCharacter(character)
	humanoid = character:WaitForChild("Humanoid")
	rootPart = character:WaitForChild("HumanoidRootPart")
	animator = humanoid:WaitForChild("Animator")

	if not spawnCFrame then
		spawnCFrame = rootPart.CFrame
	end

	if floatForce then
		floatForce:Destroy()
		floatForce = nil
	end
	floatEnabled = false

	animator.AnimationPlayed:Connect(function(track)
		if noLagEnabled then
			track:Stop(0)
		end
	end)
end

if player.Character then
	setupCharacter(player.Character)
end
player.CharacterAdded:Connect(setupCharacter)

------------------------------------------------
-- SPEED CONTROL
------------------------------------------------
RunService.Heartbeat:Connect(function()
	if humanoid and humanoid.Health > 0 and lzBoostEnabled then
		humanoid.WalkSpeed = BOOST_SPEED
	end
end)

------------------------------------------------
-- FLOAT TO BASE
------------------------------------------------
RunService.Heartbeat:Connect(function()
	if not floatEnabled or not rootPart or not spawnCFrame or not floatForce then
		return
	end

	local toBase = spawnCFrame.Position - rootPart.Position
	local distance = toBase.Magnitude

	if distance < 3 then
		floatEnabled = false
		floatForce:Destroy()
		floatForce = nil
		return
	end

	local horizontal = Vector3.new(toBase.X, 0, toBase.Z)
	if horizontal.Magnitude == 0 then return end

	local targetHeight =
		distance > NEAR_DISTANCE and FLOAT_HEIGHT_FAR or FLOAT_HEIGHT_NEAR

	local desiredY = spawnCFrame.Position.Y + targetHeight
	local upVelocity =
		rootPart.Position.Y < desiredY and LIFT_FORCE or 0

	floatForce.Velocity =
		horizontal.Unit * FLOAT_SPEED +
		Vector3.new(0, upVelocity, 0)
end)

------------------------------------------------
-- GUI
------------------------------------------------
local screenGui = Instance.new("ScreenGui", gui)
screenGui.Name = "LZ_UI"
screenGui.ResetOnSpawn = false

------------------------------------------------
-- MAIN FRAME
------------------------------------------------
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 220, 0, 270)
frame.Position = UDim2.new(0.5, -110, 0.5, -135)
frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
frame.BorderSizePixel = 0
frame.Active = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0,12)

local BORDER_COLOR = Color3.fromRGB(120,180,255)
local DARK_GRAY = Color3.fromRGB(40,40,40)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = BORDER_COLOR
stroke.Thickness = 3
stroke.Transparency = 0.15

------------------------------------------------
-- BORDER PULSE
------------------------------------------------
local pulseOut = TweenService:Create(
	stroke,
	TweenInfo.new(1.4, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
	{ Transparency = 0.75 }
)

local pulseIn = TweenService:Create(
	stroke,
	TweenInfo.new(1.4, Enum.EasingStyle.Sine, Enum.EasingDirection.In),
	{ Transparency = 0.15 }
)

pulseOut.Completed:Connect(function() pulseIn:Play() end)
pulseIn.Completed:Connect(function() pulseOut:Play() end)
pulseOut:Play()

------------------------------------------------
-- TOP BAR (DRAG ONLY)
------------------------------------------------
local topBar = Instance.new("Frame", frame)
topBar.Size = UDim2.new(1,0,0,28)
topBar.BackgroundTransparency = 1
topBar.Active = true

local title = Instance.new("TextLabel", topBar)
title.Size = UDim2.new(1,0,1,0)
title.BackgroundTransparency = 1
title.Text = "[LZ-PAID]"
title.TextColor3 = BORDER_COLOR
title.Font = Enum.Font.GothamBlack
title.TextSize = 16

------------------------------------------------
-- BUTTON CREATOR
------------------------------------------------
local padding = 10
local buttonHeight = 36

local function makeButton(text, y)
	local b = Instance.new("TextButton", frame)
	b.Size = UDim2.new(1,-padding*2,0,buttonHeight)
	b.Position = UDim2.new(0,padding,0,y)
	b.Text = text
	b.Font = Enum.Font.GothamBlack
	b.TextSize = 15
	b.BackgroundColor3 = DARK_GRAY
	b.TextColor3 = Color3.new(1,1,1)
	b.BorderSizePixel = 0
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,8)
	return b
end

------------------------------------------------
-- BUTTONS
------------------------------------------------
local y = 28 + padding

local noLagButton = makeButton("NO LAG [OFF]", y)
y += padding + buttonHeight

local boostButton = makeButton("LZ BOOST [OFF]", y)
y += padding + buttonHeight

local floatButton = makeButton("FLOAT TO BASE", y)
y += padding + buttonHeight

local instantStealButton = makeButton("LZ INSTANT STEAL", y)
y += padding + buttonHeight

local kickButton = makeButton("KICK FROM SERVER", y)

------------------------------------------------
-- BUTTON LOGIC
------------------------------------------------
noLagButton.MouseButton1Click:Connect(function()
	noLagEnabled = not noLagEnabled
	noLagButton.Text = noLagEnabled and "NO LAG [ON]" or "NO LAG [OFF]"
	noLagButton.BackgroundColor3 = noLagEnabled and BORDER_COLOR or DARK_GRAY
	noLagButton.TextColor3 = noLagEnabled and Color3.new(0,0,0) or Color3.new(1,1,1)
end)

boostButton.MouseButton1Click:Connect(function()
	lzBoostEnabled = not lzBoostEnabled
	boostButton.Text = lzBoostEnabled and "LZ BOOST [ON]" or "LZ BOOST [OFF]"
	boostButton.BackgroundColor3 = lzBoostEnabled and BORDER_COLOR or DARK_GRAY
	boostButton.TextColor3 = lzBoostEnabled and Color3.new(0,0,0) or Color3.new(1,1,1)
end)

floatButton.MouseButton1Click:Connect(function()
	floatEnabled = not floatEnabled

	if floatEnabled then
		floatButton.Text = "FLOAT TO BASE [ON]"
		floatButton.BackgroundColor3 = BORDER_COLOR
		floatButton.TextColor3 = Color3.new(0,0,0)

		floatForce = Instance.new("BodyVelocity")
		floatForce.MaxForce = Vector3.new(4e4,4e4,4e4)
		floatForce.Parent = rootPart
	else
		if floatForce then floatForce:Destroy() end
		floatButton.Text = "FLOAT TO BASE"
		floatButton.BackgroundColor3 = DARK_GRAY
		floatButton.TextColor3 = Color3.new(1,1,1)
	end
end)

------------------------------------------------
-- LZ INSTANT STEAL (INSTANT TP)
------------------------------------------------
instantStealButton.MouseButton1Click:Connect(function()
	if rootPart and spawnCFrame then
		rootPart.CFrame = spawnCFrame
	end
end)

------------------------------------------------
-- KICK FROM SERVER
------------------------------------------------
kickButton.MouseButton1Click:Connect(function()
	player:Kick("LZ KICKED YOU")
end)

------------------------------------------------
-- DRAGGING
------------------------------------------------
local dragging, dragStart, startPos

topBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
	end
end)

topBar.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(
			startPos.X.Scale, startPos.X.Offset + delta.X,
			startPos.Y.Scale, startPos.Y.Offset + delta.Y
		)
	end
end)

topBar.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)
