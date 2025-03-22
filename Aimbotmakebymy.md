local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()
local AimbotEnabled = false
local ESPEnabled = false

local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 100)
Frame.Position = UDim2.new(0, 10, 0, 10)
Frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

local AimbotButton = Instance.new("TextButton", Frame)
AimbotButton.Size = UDim2.new(1, 0, 0, 40)
AimbotButton.Position = UDim2.new(0, 0, 0, 0)
AimbotButton.Text = "Bật Aimbot"
AimbotButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

local ESPButton = Instance.new("TextButton", Frame)
ESPButton.Size = UDim2.new(1, 0, 0, 40)
ESPButton.Position = UDim2.new(0, 0, 0, 50)
ESPButton.Text = "Bật ESP"
ESPButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

function GetNearestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local rootPart = player.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)

            if onScreen then
                local distance = (Vector2.new(Mouse.X, Mouse.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end

    return closestPlayer
end

RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local targetPlayer = GetNearestPlayer()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPlayer.Character.Head.Position)
        end
    end
end)

AimbotButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    AimbotButton.Text = AimbotEnabled and "Tắt Aimbot" or "Bật Aimbot"
end)

function CreateESP(player)
    if player == LocalPlayer then return end
    local BillboardGui = Instance.new("BillboardGui", player.Character.Head)
    BillboardGui.Size = UDim2.new(0, 100, 0, 50)
    BillboardGui.StudsOffset = Vector3.new(0, 2, 0)
    BillboardGui.AlwaysOnTop = true

    local TextLabel = Instance.new("TextLabel", BillboardGui)
    TextLabel.Size = UDim2.new(1, 0, 1, 0)
    TextLabel.Text = player.Name
    TextLabel.BackgroundTransparency = 1
    TextLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
end

ESPButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    ESPButton.Text = ESPEnabled and "Tắt ESP" or "Bật ESP"

    if ESPEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("Head") then
                CreateESP(player)
            end
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("Head") then
                for _, child in pairs(player.Character.Head:GetChildren()) do
                    if child:IsA("BillboardGui") then
                        child:Destroy()
                    end
                end
            end
        end
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if ESPEnabled then
            task.wait(1)
            CreateESP(player)
        end
    end)
end)
