------------------------------------------------
-- SERVICES
------------------------------------------------
local Players = game:GetService("Players")
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
-- GUI
------------------------------------------------
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "LZ_UI"
screenGui.ResetOnSpawn = false
screenGui.Parent = gui

------------------------------------------------
-- MAIN FRAME (BLACK BOX)
------------------------------------------------
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 360, 0, 120)
frame.Position = UDim2.new(0.5, -180, 0.5, -60)
frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
frame.BorderSizePixel = 0
frame.Active = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 14)

------------------------------------------------
-- BORDER
------------------------------------------------
local BORDER_COLOR = Color3.fromRGB(120, 180, 255)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = BORDER_COLOR
stroke.Thickness = 3
stroke.Transparency = 0.15

------------------------------------------------
-- BORDER PULSE
------------------------------------------------
task.spawn(function()
	while stroke.Parent do
		TweenService:Create(
			stroke,
			TweenInfo.new(1.5, Enum.EasingStyle.Sine),
			{Transparency = 0.75}
		):Play()
		task.wait(1.5)

		TweenService:Create(
			stroke,
			TweenInfo.new(1.5, Enum.EasingStyle.Sine),
			{Transparency = 0.15}
		):Play()
		task.wait(1.5)
	end
end)

------------------------------------------------
-- TEXT LABEL (FIXED SIZE)
------------------------------------------------
local label = Instance.new("TextLabel", frame)
label.Size = UDim2.new(1, -24, 1, -24)
label.Position = UDim2.new(0, 12, 0, 12)
label.BackgroundTransparency = 1
label.TextWrapped = true
label.Text = "[ SCRIPT HAS BEEN SHUTDOWN\nFOR MORE UPDATES COMING ]"
label.TextColor3 = BORDER_COLOR
label.Font = Enum.Font.GothamBlack
label.TextSize = 18
label.TextXAlignment = Enum.TextXAlignment.Center
label.TextYAlignment = Enum.TextYAlignment.Center
