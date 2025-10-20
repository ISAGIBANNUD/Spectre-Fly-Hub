-- SpectreFlyGui - LocalScript
-- Criado por ChatGPT (Spectre Edition)
-- Permite voar e ajustar velocidade com + e -

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local character, hrp, humanoid
local flying = false
local flySpeed = 50
local moveDir = Vector3.zero
local flyConnection

-- Função para obter o personagem e partes
local function refreshCharacter()
	character = player.Character or player.CharacterAdded:Wait()
	hrp = character:WaitForChild("HumanoidRootPart")
	humanoid = character:WaitForChild("Humanoid")
end

refreshCharacter()

player.CharacterAdded:Connect(function()
	wait(1)
	refreshCharacter()
end)

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.Name = "SpectreFlyGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 120)
frame.Position = UDim2.new(0.5, -100, 0.7, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0
frame.Parent = gui
local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "🌀 Spectre Fly GUI"
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.fromRGB(0, 200, 255)
title.TextScaled = true

-- Botão principal Fly
local flyButton = Instance.new("TextButton", frame)
flyButton.Size = UDim2.new(0, 180, 0, 35)
flyButton.Position = UDim2.new(0, 10, 0, 40)
flyButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
flyButton.TextColor3 = Color3.new(1, 1, 1)
flyButton.Font = Enum.Font.GothamBold
flyButton.TextScaled = true
flyButton.Text = "Fly: OFF"
local flyCorner = Instance.new("UICorner", flyButton)
flyCorner.CornerRadius = UDim.new(0, 8)

-- Botões + e -
local minus = Instance.new("TextButton", frame)
minus.Size = UDim2.new(0, 60, 0, 30)
minus.Position = UDim2.new(0, 10, 0, 80)
minus.Text = "-"
minus.Font = Enum.Font.GothamBold
minus.TextScaled = true
minus.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
minus.TextColor3 = Color3.new(1, 1, 1)
local cornerMinus = Instance.new("UICorner", minus)
cornerMinus.CornerRadius = UDim.new(0, 8)

local plus = Instance.new("TextButton", frame)
plus.Size = UDim2.new(0, 60, 0, 30)
plus.Position = UDim2.new(1, -70, 0, 80)
plus.Text = "+"
plus.Font = Enum.Font.GothamBold
plus.TextScaled = true
plus.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
plus.TextColor3 = Color3.new(1, 1, 1)
local cornerPlus = Instance.new("UICorner", plus)
cornerPlus.CornerRadius = UDim.new(0, 8)

local speedLabel = Instance.new("TextLabel", frame)
speedLabel.Size = UDim2.new(0, 60, 0, 30)
speedLabel.Position = UDim2.new(0.5, -30, 0, 80)
speedLabel.BackgroundTransparency = 1
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextScaled = true
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.Text = tostring(flySpeed)

-- ===== FLY LÓGICA =====
local keys = {W = false, A = false, S = false, D = false, Space = false, Shift = false}

local function updateMoveDir()
	local cam = workspace.CurrentCamera
	if not cam then return Vector3.zero end
	local forward = cam.CFrame.LookVector
	local right = cam.CFrame.RightVector
	local move = Vector3.zero
	if keys.W then move += forward end
	if keys.S then move -= forward end
	if keys.A then move -= right end
	if keys.D then move += right end
	if keys.Space then move += Vector3.new(0,1,0) end
	if keys.Shift then move -= Vector3.new(0,1,0) end
	if move.Magnitude > 0 then
		moveDir = move.Unit
	else
		moveDir = Vector3.zero
	end
end

UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.W then keys.W = true end
	if input.KeyCode == Enum.KeyCode.S then keys.S = true end
	if input.KeyCode == Enum.KeyCode.A then keys.A = true end
	if input.KeyCode == Enum.KeyCode.D then keys.D = true end
	if input.KeyCode == Enum.KeyCode.Space then keys.Space = true end
	if input.KeyCode == Enum.KeyCode.LeftShift then keys.Shift = true end
	updateMoveDir()
end)

UserInputService.InputEnded:Connect(function(input, gp)
	if input.KeyCode == Enum.KeyCode.W then keys.W = false end
	if input.KeyCode == Enum.KeyCode.S then keys.S = false end
	if input.KeyCode == Enum.KeyCode.A then keys.A = false end
	if input.KeyCode == Enum.KeyCode.D then keys.D = false end
	if input.KeyCode == Enum.KeyCode.Space then keys.Space = false end
	if input.KeyCode == Enum.KeyCode.LeftShift then keys.Shift = false end
	updateMoveDir()
end)

local function startFly()
	if flyConnection then return end
	humanoid.PlatformStand = true
	flyConnection = RunService.RenderStepped:Connect(function()
		if not hrp then return end
		if moveDir.Magnitude > 0 then
			hrp.Velocity = moveDir * flySpeed
		else
			hrp.Velocity = Vector3.zero
		end
	end)
end

local function stopFly()
	if flyConnection then
		flyConnection:Disconnect()
		flyConnection = nil
	end
	if humanoid then
		humanoid.PlatformStand = false
	end
	if hrp then
		hrp.Velocity = Vector3.zero
	end
end

flyButton.MouseButton1Click:Connect(function()
	flying = not flying
	if flying then
		flyButton.Text = "Fly: ON"
		startFly()
	else
		flyButton.Text = "Fly: OFF"
		stopFly()
	end
end)

-- ===== Ajuste de velocidade =====
plus.MouseButton1Click:Connect(function()
	flySpeed += 10
	speedLabel.Text = tostring(flySpeed)
end)

minus.MouseButton1Click:Connect(function()
	flySpeed = math.max(10, flySpeed - 10)
	speedLabel.Text = tostring(flySpeed)
end)
