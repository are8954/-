-- ใส่ไว้ใน StarterPlayer -> StarterPlayerScripts (LocalScript)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- === [ ตัวแปรหลัก / SETTINGS ] ===
local Settings = {
	AimbotEnabled = true,
	AimKey = Enum.UserInputType.MouseButton2, -- เปลี่ยนเริ่มต้นเป็น "คลิกขวา" เพื่อให้ใช้งานง่ายขึ้น
	FOVEnabled = true,
	FOVRadius = 150,
	ESPEnabled = true,
	TeamCheck = true, -- เริ่มต้นเปิดระบบเช็คทีมตรงข้าม (true = ล็อกเฉพาะทีมตรงข้าม, false = ล็อกทุกคน)
}

-- === [ ระบบวงกลม FOV & ESP ] ===
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1.5
FOVCircle.Filled = false
FOVCircle.Color = Color3.fromRGB(0, 255, 255)

local ESPObjects = {}
local function createESP(player)
	if ESPObjects[player] then return end
	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Filled = false
	box.Color = Color3.fromRGB(255, 0, 0)
	ESPObjects[player] = box
end

local function removeESP(player)
	if ESPObjects[player] then
		ESPObjects[player]:Remove()
		ESPObjects[player] = nil
	end
end

Players.PlayerAdded:Connect(createESP)
Players.PlayerRemoving:Connect(removeESP)
for _, p in ipairs(Players:GetPlayers()) do
	if p ~= LocalPlayer then createESP(p) end
end

-- === [ การสร้างหน้าต่างหน้าจอ GUI ] ===
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AimbotMenuGui"
ScreenGui.ResetOnSpawn = false
-- พยายามใส่ไว้ใน CoreGui เพื่อไม่ให้หลุดตอนตัวละครตาย (ถ้าเปิดสิทธิ์ Studio ไว้) ไม่เช่นนั้นจะย้ายไป PlayerGui อัตโนมัติ
pcall(function() ScreenGui.Parent = CoreGui end)
if not ScreenGui.Parent then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

-- หน้าต่างหลัก (Main Frame)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 220, 0, 280)
MainFrame.Position = UDim2.new(0.05, 0, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true -- สามารถลากย้ายตำแหน่งบนหน้าจอได้
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

-- หัวข้อเมนู (Title)
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Title.Text = "🎯 AIMBOT MENU (TEAM CHECK)"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 16
Title.BorderSizePixel = 0
Title.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = Title

-- ฟังก์ชันสร้างปุ่มเปิด/ปิด (Toggle Button Setup)
local function createToggle(name, text, default, position, callback)
	local button = Instance.new("TextButton")
	button.Name = name
	button.Size = UDim2.new(0.9, 0, 0, 35)
	button.Position = position
	button.Font = Enum.Font.SourceSans
	button.TextSize = 16
	button.BorderSizePixel = 0
	button.Parent = MainFrame
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 5)
	corner.Parent = button

	local state = default
	local function updateDisplay()
		if state then
			button.BackgroundColor3 = Color3.fromRGB(46, 204, 113) -- สีเขียว
			button.Text = text .. ": ON"
			button.TextColor3 = Color3.fromRGB(255, 255, 255)
		else
			button.BackgroundColor3 = Color3.fromRGB(231, 76, 60) -- สีแดง
			button.Text = text .. ": OFF"
			button.TextColor3 = Color3.fromRGB(255, 255, 255)
		end
	end
	
	button.MouseButton1Click:Connect(function()
		state = not state
		updateDisplay()
		callback(state)
	end)
	
	updateDisplay()
end

-- สร้างปุ่มต่างๆ ลงบน GUI
createToggle("AimbotToggle", "Aimbot Lock", Settings.AimbotEnabled, UDim2.new(0.05, 0, 0, 50), function(v) Settings.AimbotEnabled = v end)
createToggle("TeamCheckToggle", "Team Check", Settings.TeamCheck, UDim2.new(0.05, 0, 0, 95), function(v) Settings.TeamCheck = v end)
createToggle("FOVToggle", "Show FOV Circle", Settings.FOVEnabled, UDim2.new(0.05, 0, 0, 140), function(v) Settings.FOVEnabled = v end)
createToggle("ESPToggle", "ESP Box (Enemy)", Settings.ESPEnabled, UDim2.new(0.05, 0, 0, 185), function(v) Settings.ESPEnabled = v end)

-- ปุ่มสำหรับเพิ่ม/ลดขนาด FOV
local FOVControl = Instance.new("Frame")
FOVControl.Size = UDim2.new(0.9, 0, 0, 35)
FOVControl.Position = UDim2.new(0.05, 0, 0, 230)
FOVControl.BackgroundTransparency = 1
FOVControl.Parent = MainFrame

local FOVLabel = Instance.new("TextLabel")
FOVLabel.Size = UDim2.new(0.5, 0, 1, 0)
FOVLabel.Text = "FOV: " .. Settings.FOVRadius
FOVLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
FOVLabel.Font = Enum.Font.SourceSans
FOVLabel.TextSize = 14
FOVLabel.BackgroundTransparency = 1
FOVLabel.Parent = FOVControl

local BtnMinus = Instance.new("TextButton")
BtnMinus.Size = UDim2.new(0.2, 0, 1, 0)
BtnMinus.Position = UDim2.new(0.5, 0, 0, 0)
BtnMinus.Text = "-"
BtnMinus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
BtnMinus.TextColor3 = Color3.fromRGB(255, 255, 255)
BtnMinus.Parent = FOVControl

local BtnPlus = Instance.new("TextButton")
BtnPlus.Size = UDim2.new(0.2, 0, 1, 0)
BtnPlus.Position = UDim2.new(0.75, 0, 0, 0)
BtnPlus.Text = "+"
BtnPlus.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
BtnPlus.TextColor3 = Color3.fromRGB(255, 255, 255)
BtnPlus.Parent = FOVControl

BtnMinus.MouseButton1Click:Connect(function()
	Settings.FOVRadius = math.max(10, Settings.FOVRadius - 25)
	FOVLabel.Text = "FOV: " .. Settings.FOVRadius
end)

BtnPlus.MouseButton1Click:Connect(function()
	Settings.FOVRadius = math.min(800, Settings.FOVRadius + 25)
	FOVLabel.Text = "FOV: " .. Settings.FOVRadius
end)

-- ปุ่มซ่อน/แสดงเมนู (กด Insert บนคีย์บอร์ด)
UserInputService.InputBegan:Connect(function(input, processed)
	if not processed and input.KeyCode == Enum.KeyCode.Insert then
		MainFrame.Visible = not MainFrame.Visible
	end
end)

-- === [ ฟังก์ชันหาเป้าหมายที่ตรงเงื่อนไข + เช็คทีม ] ===
local function getClosestPlayerToMouse()
	local target = nil
	local closestDistance = Settings.FOVEnabled and Settings.FOVRadius or math.huge
	local mousePos = UserInputService:GetMouseLocation()

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character then
			-- เพิ่มระบบ Team Check: ถ้าเปิดใช้งานอยู่ และเป็นทีมเดียวกัน -> ข้ามไปเลย ไม่ล็อกเป้า
			if Settings.TeamCheck and player.Team == LocalPlayer.Team then
				continue
			end
			
			local char = player.Character
			local head = char:FindFirstChild("Head")
			local humanoid = char:FindFirstChildOfClass("Humanoid")
			
			if head and humanoid and humanoid.Health > 0 then
				local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
				if onScreen then
					local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
					if distance < closestDistance then
						closestDistance = distance
						target = player
					end
				end
			end
		end
	end
	return target
end

-- === [ MAIN LOOP ทำงานทุกเฟรม ] ===
RunService.RenderStepped:Connect(function()
	local mousePos = UserInputService:GetMouseLocation()
	
	-- อัปเดต FOV
	FOVCircle.Position = mousePos
	FOVCircle.Visible = Settings.FOVEnabled
	FOVCircle.Radius = Settings.FOVRadius
	
	-- อัปเดตกรอบ ESP สี่เหลี่ยมรอบตัวศัตรู
	for player, box in pairs(ESPObjects) do
		local isEnemy = not Settings.TeamCheck or (player.Team ~= LocalPlayer.Team)
		
		if Settings.ESPEnabled and isEnemy and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChildOfClass("Humanoid") and player.Character:FindFirstChildOfClass("Humanoid").Health > 0 then
			local hrp = player.Character.HumanoidRootPart
			local hrpPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
			
			if onScreen then
				local scale = 1000 / hrpPos.Z
				box.Size = Vector2.new(scale * 2.5, scale * 3.5)
				box.Position = Vector2.new(hrpPos.X - box.Size.X / 2, hrpPos.Y - box.Size.Y / 2)
				box.Visible = true
			else
				box.Visible = false
			end
		else
			box.Visible = false
		end
	end
	
	-- ล็อกเป้าเมื่อกด "คลิกขวา" ค้างไว้
	if Settings.AimbotEnabled and (UserInputService:IsKeyDown(Settings.AimKey) or UserInputService:IsMouseButtonPressed(Settings.AimKey)) then
		local target = getClosestPlayerToMouse()
		if target and target.Character and target.Character:FindFirstChild("Head") then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
		end
	end
end)
