-- SERVIÇOS
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local playerStats = require(localPlayer:WaitForChild("plrStats"))
local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- CONFIGURAÇÕES
getgenv().settings = {
    enabled = false,
    max_distance = 20,
    delay = 0.2,
    move_forward_enabled = false,
    move_speed = 2 -- velocidade mais lenta para parecer natural
}

-- DANO
function current_damage()
    local damage = 0
    for i, v in next, playerStats.wpnStats do
        if i == "Dmg" and v > damage then
            damage = v
        end
    end
    return damage
end

-- DETECTAR INIMIGOS
function getAliveEnemies()
    local enemies = {}
    for _, v in ipairs(Workspace:GetChildren()) do
        if v:IsA("Model") and v ~= localPlayer.Character and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            table.insert(enemies, v)
        end
    end
    return enemies
end

-- Kill Aura com espera aleatória e simulação leve
task.spawn(function()
    while true do
        if getgenv().settings.enabled then
            character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
            humanoidRootPart = character:FindFirstChild("HumanoidRootPart")

            local enemies = getAliveEnemies()

            for _, enemy in ipairs(enemies) do
                if enemy and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                    local distance = (enemy:GetPivot().Position - humanoidRootPart.Position).Magnitude
                    if distance <= getgenv().settings.max_distance then
                        -- Pequeno delay aleatório para simular atraso humano
                        task.wait(math.random(8, 15)/100)
                        ReplicatedStorage:WaitForChild("remotes"):WaitForChild("onHit"):FireServer(enemy.Humanoid, current_damage(), {}, 0)
                    end
                end
            end
        end
        task.wait(getgenv().settings.delay)
    end
end)

-- Movimento contínuo para frente simulando caminhada
RunService.RenderStepped:Connect(function()
    if getgenv().settings.move_forward_enabled then
        if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
            localPlayer.Character:TranslateBy(localPlayer.Character.HumanoidRootPart.CFrame.LookVector * getgenv().settings.move_speed * RunService.RenderStepped:Wait())
        end
    end
end)

-- UI BÁSICA
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "SafeKillAuraUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 180)
frame.Position = UDim2.new(0, 50, 0, 200)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "Safe Kill Aura"
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 14

local toggle = Instance.new("TextButton", frame)
toggle.Size = UDim2.new(1, -20, 0, 30)
toggle.Position = UDim2.new(0, 10, 0, 40)
toggle.Text = "Kill Aura: OFF"
toggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.MouseButton1Click:Connect(function()
    getgenv().settings.enabled = not getgenv().settings.enabled
    toggle.Text = "Kill Aura: " .. (getgenv().settings.enabled and "ON" or "OFF")
end)

local moveToggle = Instance.new("TextButton", frame)
moveToggle.Size = UDim2.new(1, -20, 0, 30)
moveToggle.Position = UDim2.new(0, 10, 0, 80)
moveToggle.Text = "Avançar Fases: OFF"
moveToggle.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
moveToggle.TextColor3 = Color3.new(1, 1, 1)
moveToggle.MouseButton1Click:Connect(function()
    getgenv().settings.move_forward_enabled = not getgenv().settings.move_forward_enabled
    moveToggle.Text = "Avançar Fases: " .. (getgenv().settings.move_forward_enabled and "ON" or "OFF")
end)
