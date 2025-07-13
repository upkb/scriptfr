-- Loading Screen Setup
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "Loading Script.."
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local loadingFrame = Instance.new("Frame")
loadingFrame.Size = UDim2.new(0, 500, 0, 150)
loadingFrame.AnchorPoint = Vector2.new(0.5, 0.5)
loadingFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
loadingFrame.BackgroundColor3 = Color3.new(0, 0, 0)
loadingFrame.BorderSizePixel = 0
loadingFrame.Parent = screenGui

-- Animated Stars
local starCount = 150
for i = 1, starCount do
    local star = Instance.new("Frame")
    star.Size = UDim2.new(0, 2, 0, 2)
    star.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    star.BorderSizePixel = 0
    star.AnchorPoint = Vector2.new(0.5, 0.5)
    star.Position = UDim2.new(math.random(), 0, math.random(), 0)
    star.BackgroundTransparency = 0.2 + math.random() * 0.8
    star.Rotation = math.random(0, 360)
    star.Parent = loadingFrame
end

-- Loading Text
local loadingText = Instance.new("TextLabel")
loadingText.Size = UDim2.new(1, 0, 0, 50)
loadingText.Position = UDim2.new(0, 0, 0, 15)
loadingText.BackgroundTransparency = 1
loadingText.TextColor3 = Color3.fromRGB(255, 255, 255)
loadingText.Font = Enum.Font.GothamBold
loadingText.TextSize = 36
loadingText.Text = "Loading Script.."
loadingText.Parent = loadingFrame

-- Progress Bar
local barBackground = Instance.new("Frame")
barBackground.Size = UDim2.new(0.8, 0, 0, 30)
barBackground.Position = UDim2.new(0.1, 0, 0, 95)
barBackground.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
barBackground.BorderSizePixel = 0
barBackground.Parent = loadingFrame

local barFill = Instance.new("Frame")
barFill.Size = UDim2.new(0, 0, 1, 0)
barFill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
barFill.BorderSizePixel = 0
barFill.Parent = barBackground

local tweenService = game:GetService("TweenService")
local tweenInfo = TweenInfo.new(3, Enum.EasingStyle.Linear)
local tween = tweenService:Create(barFill, tweenInfo, {Size = UDim2.new(1, 0, 1, 0)})
tween:Play()

-- Wait for the loading bar to complete
tween.Completed:Wait()

-- Fade Out Loading Screen
local fadeOutTween = tweenService:Create(loadingFrame, TweenInfo.new(0.5, Enum.EasingStyle.Linear), {BackgroundTransparency = 1})
fadeOutTween:Play()
fadeOutTween.Completed:Wait()

screenGui:Destroy()

-- Main Script Injection
local scriptContent = [[
    local StarterGui = game:GetService("StarterGui")

-- SETTINGS
getgenv().AimEnabled = true
getgenv().AimKey = Enum.KeyCode.Q
getgenv().DisableKey = Enum.KeyCode.P
getgenv().TeleportKey = Enum.KeyCode.T
getgenv().PreferredParts = {"Head", "UpperTorso", "HumanoidRootPart"}
getgenv().AutoPrediction = true
getgenv().ManualPrediction = 0.125
getgenv().Smoothness = nil -- nil = hard lock; number = smooth

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

local Locked = false
local Target = nil

-- Get player ping
local function getPing()
    local stats = LocalPlayer:GetNetworkPing()
    return math.clamp(stats, 0.03, 0.3)
end

-- Get preferred part
local function getValidPart(char)
    for _, partName in ipairs(getgenv().PreferredParts) do
        local part = char:FindFirstChild(partName)
        if part then return part end
    end
    return nil
end

-- Get closest player to mouse cursor
local function getClosestToCursor()
    local closestPlayer = nil
    local closestDistance = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local part = getValidPart(player.Character)
            if part then
                local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local dist = (Vector2.new(Mouse.X, Mouse.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                    if dist < closestDistance then
                        closestDistance = dist
                        closestPlayer = player
                    end
                end
            end
        end
    end

    return closestPlayer
end

-- KEYBINDS
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end

    if input.KeyCode == getgenv().AimKey then
        if getgenv().AimEnabled then
            if Locked then
                Locked = false
                Target = nil
            else
                local closest = getClosestToCursor()
                if closest then
                    Locked = true
                    Target = closest
                end
            end
        end

    elseif input.KeyCode == getgenv().DisableKey then
        getgenv().AimEnabled = false
        Locked = false
        Target = nil

    elseif input.KeyCode == getgenv().TeleportKey then
        if getgenv().AimEnabled and Locked and Target and Target.Character then
            local targetHRP = Target.Character:FindFirstChild("HumanoidRootPart")
            local localHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

            if targetHRP and localHRP then
                local backOffset = -targetHRP.CFrame.LookVector * 5
                local targetPosition = targetHRP.Position + backOffset + Vector3.new(0, 1.5, 0)
                localHRP.CFrame = CFrame.new(targetPosition, targetHRP.Position)
            end
        end
    end
end)

-- AIMLOOP
RunService.RenderStepped:Connect(function()
    if getgenv().AimEnabled and Locked and Target and Target.Character then
        local part = getValidPart(Target.Character)
        if part then
            local predictionTime = getgenv().AutoPrediction and getPing() or getgenv().ManualPrediction
            local predictedPos = part.Position + part.Velocity * predictionTime
            local aimCFrame = CFrame.new(Camera.CFrame.Position, predictedPos)

            if getgenv().Smoothness == nil then
                Camera.CFrame = aimCFrame
            else
                Camera.CFrame = Camera.CFrame:Lerp(aimCFrame, getgenv().Smoothness)
            end
        end
    end
end)

-- SPEED SCRIPT
local speed = 270
local humanoid
local speedEnabled = false
local speedConnection

local function enforceSpeed()
    if humanoid and speedEnabled and humanoid.WalkSpeed ~= speed then
        humanoid.WalkSpeed = speed
    end
end

local function onCharacterAdded(character)
    humanoid = character:WaitForChild("Humanoid")
    if speedEnabled then
        humanoid.WalkSpeed = speed
        speedConnection = humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(enforceSpeed)
    end
end

local function toggleSpeed()
    speedEnabled = not speedEnabled
    if humanoid then
        if speedEnabled then
            humanoid.WalkSpeed = speed
            speedConnection = humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(enforceSpeed)
        else
            if speedConnection then
                speedConnection:Disconnect()
                speedConnection = nil
            end
            humanoid.WalkSpeed = 16
        end
    end
end

UIS.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Z then
        toggleSpeed()
    end
end)

LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
if LocalPlayer.Character then
    onCharacterAdded(LocalPlayer.Character)
end

-- JUMP BOOST
local JUMP_KEY = Enum.KeyCode.X
local BOOST_POWER = 200
local DEFAULT_POWER = 50
local jumpEnabled = false
local jumpConnection = nil

local function enforceJump()
    if humanoid and jumpEnabled and humanoid.JumpPower ~= BOOST_POWER then
        humanoid.JumpPower = BOOST_POWER
    end
end

local function toggleJump()
    jumpEnabled = not jumpEnabled
    if humanoid then
        if jumpEnabled then
            humanoid.JumpPower = BOOST_POWER
            jumpConnection = humanoid:GetPropertyChangedSignal("JumpPower"):Connect(enforceJump)
        else
            if jumpConnection then
                jumpConnection:Disconnect()
                jumpConnection = nil
            end
            humanoid.JumpPower = DEFAULT_POWER
        end
    end
end

UIS.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == JUMP_KEY then
        toggleJump()
    end
end)

LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
if LocalPlayer.Character then
    onCharacterAdded(LocalPlayer.Character)
end

]]
loadstring(scriptContent)()
