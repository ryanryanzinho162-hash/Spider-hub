local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local HRP = Character:WaitForChild("HumanoidRootPart")

local speedEnabled, jumpEnabled, noclipEnabled, flyEnabled, autoCollectEnabled = false, false, false, false, false
local walkSpeed, flySpeed = 16, 50

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.Name = "LennonHubClone"

local Menu = Instance.new("Frame")
Menu.Size = UDim2.new(0, 350, 0, 500)
Menu.Position = UDim2.new(0.5, -175, 0.5, -250)
Menu.BackgroundColor3 = Color3.fromRGB(35,35,35)
Menu.BorderSizePixel = 0
Menu.Visible = false
Menu.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Text = "Lennon Hub Clone"
Title.Size = UDim2.new(1,0,0,50)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 24
Title.Parent = Menu

local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    Menu.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
                              startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Menu.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

Title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

local OpenButton = Instance.new("TextButton")
OpenButton.Size = UDim2.new(0, 40, 0, 40)
OpenButton.Position = UDim2.new(1, -50, 0, 10)
OpenButton.AnchorPoint = Vector2.new(1,0)
OpenButton.BackgroundColor3 = Color3.fromRGB(200,0,0)
OpenButton.Text = "☰"
OpenButton.TextColor3 = Color3.fromRGB(255,255,255)
OpenButton.Font = Enum.Font.SourceSansBold
OpenButton.TextSize = 24
OpenButton.Parent = ScreenGui

OpenButton.MouseButton1Click:Connect(function()
    Menu.Visible = not Menu.Visible
end)

local function createButton(name, callback, posY)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1,-20,0,40)
    button.Position = UDim2.new(0,10,0,posY)
    button.BackgroundColor3 = Color3.fromRGB(50,50,50)
    button.TextColor3 = Color3.fromRGB(255,255,255)
    button.Text = name
    button.Font = Enum.Font.SourceSans
    button.TextSize = 20
    button.Parent = Menu
    button.MouseButton1Click:Connect(callback)
end

createButton("Toggle Speed", function() speedEnabled = not speedEnabled end, 60)
createButton("Set WalkSpeed", function()
    local input = tonumber(LocalPlayer.PlayerGui:FindFirstChild("WalkSpeedInput") and LocalPlayer.PlayerGui.WalkSpeedInput.Text)
    if input then walkSpeed = input end
end, 110)
createButton("Toggle Jump", function() jumpEnabled = not jumpEnabled end, 160)
createButton("Toggle Noclip", function() noclipEnabled = not noclipEnabled end, 210)
createButton("Toggle Fly", function() flyEnabled = not flyEnabled end, 260)
createButton("Set Fly Speed", function()
    local input = tonumber(LocalPlayer.PlayerGui:FindFirstChild("FlySpeedInput") and LocalPlayer.PlayerGui.FlySpeedInput.Text)
    if input then flySpeed = input end
end, 310)
createButton("Toggle Auto Collect", function() autoCollectEnabled = not autoCollectEnabled end, 360)

RunService.RenderStepped:Connect(function()
    Humanoid.WalkSpeed = speedEnabled and walkSpeed or 16
    Humanoid.JumpPower = jumpEnabled and 100 or 50
    for _, part in pairs(Character:GetChildren()) do
        if part:IsA("BasePart") then
            part.CanCollide = not noclipEnabled
        end
    end
    if flyEnabled then
        local moveVector = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then moveVector = moveVector + workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then moveVector = moveVector - workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then moveVector = moveVector - workspace.CurrentCamera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then moveVector = moveVector + workspace.CurrentCamera.CFrame.RightVector end
        HRP.Velocity = moveVector.Unit * flySpeed
    end
    if autoCollectEnabled then
        for _, money in pairs(workspace:GetDescendants()) do
            if money.Name == "MoneyPart" and money:IsA("BasePart") then
                HRP.CFrame = CFrame.new(money.Position + Vector3.new(0,3,0))
            end
        end
    end
end)
