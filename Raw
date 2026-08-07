-- Roblox Main Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

-- Script Configuration
local Settings = {
    Enabled = true,
    TeamCheck = true,
    WallCheck = true,
    FOV_Radius = 150,
    FOV_Color = Color3.fromRGB(0, 255, 150), -- Mint Green
    FOV_Visible = true,
    LockKey = Enum.UserInputType.MouseButton2
}

-- Create FOV Circle (Drawing API)
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Settings.FOV_Color
FOVCircle.Thickness = 1.5
FOVCircle.NumSides = 64
FOVCircle.Radius = Settings.FOV_Radius
FOVCircle.Filled = false
FOVCircle.Visible = Settings.FOV_Visible

-- 1. GUI System Setup
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "IndyHubAimbot"
ScreenGui.ResetOnSpawn = false
pcall(function() ScreenGui.Parent = CoreGui end) or pcall(function() ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end)

-- Square Menu Toggle Button
local MenuToggleButton = Instance.new("TextButton")
MenuToggleButton.Size = UDim2.new(0, 45, 0, 45)
MenuToggleButton.Position = UDim2.new(0.02, 0, 0.2, 0)
MenuToggleButton.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
MenuToggleButton.BorderSizePixel = 2
MenuToggleButton.BorderColor3 = Color3.fromRGB(0, 255, 150)
MenuToggleButton.Text = "MENU"
MenuToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MenuToggleButton.Font = Enum.Font.SourceSansBold
MenuToggleButton.TextSize = 14
MenuToggleButton.Parent = ScreenGui

-- Main Settings Window
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 210)
MainFrame.Position = UDim2.new(0.06, 0, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

-- Toggle Menu Visibility
MenuToggleButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Window Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
Title.Text = "  AIMBOT SETTINGS"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 16
Title.Parent = MainFrame

-- Aimbot State Button
local ToggleAim = Instance.new("TextButton")
ToggleAim.Size = UDim2.new(0.9, 0, 0, 30)
ToggleAim.Position = UDim2.new(0.05, 0, 0.22, 0)
ToggleAim.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
ToggleAim.Text = "Aimbot: ON"
ToggleAim.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleAim.Font = Enum.Font.SourceSansBold
ToggleAim.TextSize = 14
ToggleAim.Parent = MainFrame

ToggleAim.MouseButton1Click:Connect(function()
    Settings.Enabled = not Settings.Enabled
    if Settings.Enabled then
        ToggleAim.Text = "Aimbot: ON"
        ToggleAim.BackgroundColor3 = Color3.fromRGB(0, 180, 100)
    else
        ToggleAim.Text = "Aimbot: OFF"
        ToggleAim.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    end
end)

-- FOV Size Indicator
local FOVLabel = Instance.new("TextLabel")
FOVLabel.Size = UDim2.new(0.9, 0, 0, 20)
FOVLabel.Position = UDim2.new(0.05, 0, 0.45, 0)
FOVLabel.BackgroundTransparency = 1
FOVLabel.Text = "FOV Size: " .. Settings.FOV_Radius
FOVLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
FOVLabel.Font = Enum.Font.SourceSans
FOVLabel.TextSize = 14
FOVLabel.TextXAlignment = Enum.TextXAlignment.Left
FOVLabel.Parent = MainFrame

-- FOV Slider Background
local SliderFrame = Instance.new("Frame")
SliderFrame.Size = UDim2.new(0.9, 0, 0, 10)
SliderFrame.Position = UDim2.new(0.05, 0, 0.58, 0)
SliderFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
SliderFrame.Parent = MainFrame

-- FOV Slider Button
local SliderButton = Instance.new("TextButton")
SliderButton.Size = UDim2.new(0, 20, 0, 20)
SliderButton.Position = UDim2.new(0.3, -10, 0.5, -10)
SliderButton.BackgroundColor3 = Color3.fromRGB(0, 255, 150)
SliderButton.Text = ""
SliderButton.Parent = SliderFrame

-- Credit Label
local CreditLabel = Instance.new("TextLabel")
CreditLabel.Size = UDim2.new(1, 0, 0, 25)
CreditLabel.Position = UDim2.new(0, 0, 0.85, 0)
CreditLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
CreditLabel.BorderSizePixel = 0
CreditLabel.Text = "by IndyHUB"
CreditLabel.TextColor3 = Color3.fromRGB(0, 255, 150)
CreditLabel.Font = Enum.Font.SourceSansItalic
CreditLabel.TextSize = 14
CreditLabel.Parent = MainFrame

-- Slider Logic
local Interacting = false
SliderButton.MouseButton1Down:Connect(function() Interacting = true end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then Interacting = false end end)

RunService.RenderStepped:Connect(function()
    if Interacting then
        local MousePos = UserInputService:GetMouseLocation()
        local RelativeX = math.clamp((MousePos.X - SliderFrame.AbsolutePosition.X) / SliderFrame.AbsoluteSize.X, 0, 1)
        SliderButton.Position = UDim2.new(RelativeX, -10, 0.5, -10)
        
        Settings.FOV_Radius = math.floor(10 + (RelativeX * 490))
        FOVLabel.Text = "FOV Size: " .. Settings.FOV_Radius
    end
    
    if Settings.FOV_Visible and Settings.Enabled then
        FOVCircle.Visible = true
        FOVCircle.Radius = Settings.FOV_Radius
        FOVCircle.Position = UserInputService:GetMouseLocation()
    else
        FOVCircle.Visible = false
    end
end)

-- 2. Raycast Obstruction Detector (Wall Check)
local function isVisible(targetPart, character)
    if not Settings.WallCheck then return true end
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, character}
    raycastParams.IgnoreWater = true
    
    local origin = Camera.CFrame.Position
    local direction = targetPart.Position - origin
    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    
    if not raycastResult then
        return true
    end
    return false
end

-- 3. Closest Target Finder
local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge
    local mousePos = UserInputService:GetMouseLocation()

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if not Settings.TeamCheck or player.Team ~= LocalPlayer.Team then
                local character = player.Character
                if character and character:FindFirstChild("Head") and character:FindFirstChildOfClass("Humanoid") and character:FindFirstChildOfClass("Humanoid").Health > 0 then
                    
                    local headPos, onScreen = Camera:WorldToViewportPoint(character.Head.Position)
                    
                    if onScreen then
                        local distance = (Vector2.new(headPos.X, headPos.Y) - mousePos).Magnitude
                        
                        if distance < Settings.FOV_Radius and distance < shortestDistance then
                            if isVisible(character.Head, character) then
                                shortestDistance = distance
                                closestPlayer = character.Head
                            end
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- 4. Main Aimbot Loop
RunService.RenderStepped:Connect(function()
    local isKeyDown = UserInputService:IsMouseButtonPressed(Settings.LockKey)
    
    if Settings.Enabled and isKeyDown then
        local target = getClosestPlayer()
        if target then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
        end
    end
end)
