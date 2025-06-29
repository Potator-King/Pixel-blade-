-- SERVIÇOS
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local localPlayer = Players.LocalPlayer
local playerStats = require(localPlayer:WaitForChild("plrStats"))
local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- CONFIGURAÇÕES INICIAIS
getgenv().settings = {
    enabled = false,
    max_distance = 25,
    delay = 0.1,
    move_distance = 50,
    move_speed = 100
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

-- DETECTAR INIMIGOS VIVOS
function getAliveEnemies()
    local enemies = {}
    for _, v in ipairs(Workspace:GetChildren()) do
        if v:IsA("Model") and v ~= localPlayer.Character and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            table.insert(enemies, v)
        end
    end
    return enemies
end

-- VOAR PRA FRENTE
function moveForward()
    local bv = Instance.new("BodyVelocity")
    bv.Velocity = humanoidRootPart.CFrame.LookVector * getgenv().settings.move_speed
    bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bv.Parent = humanoidRootPart
    game.Debris:AddItem(bv, 1)
end

-- LOOP DE COMBATE E PROGRESSÃO
spawn(function()
    while true do
        if getgenv().settings.enabled then
            character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
            humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
            local enemies = getAliveEnemies()
            if #enemies > 0 then
                for _, enemy in ipairs(enemies) do
                    if enemy and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                        local distance = (enemy:GetPivot().Position - humanoidRootPart.Position).Magnitude
                        if distance <= getgenv().settings.max_distance then
                            ReplicatedStorage:WaitForChild("remotes"):WaitForChild("onHit"):FireServer(enemy.Humanoid, current_damage(), {}, 0)
                        end
                    end
                end
            else
                moveForward()
                task.wait(1.5)
            end
        end
        task.wait(getgenv().settings.delay)
    end
end)

-- ======================
-- UI
-- ======================

local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "KillAuraUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 260)
frame.Position = UDim2.new(0, 50, 0, 200)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "Kill Aura - Pixel Blade"
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 14
title.Font = Enum.Font.SourceSansBold

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

local maxDist = Instance.new("TextBox", frame)
maxDist.Size = UDim2.new(1, -20, 0, 25)
maxDist.Position = UDim2.new(0, 10, 0, 80)
maxDist.PlaceholderText = "Distância de ataque (atual: 25)"
maxDist.Text = ""
maxDist.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
maxDist.TextColor3 = Color3.new(1, 1, 1)
maxDist.FocusLost:Connect(function()
    local val = tonumber(maxDist.Text)
    if val then
        getgenv().settings.max_distance = val
    end
end)

local delayBox = Instance.new("TextBox", frame)
delayBox.Size = UDim2.new(1, -20, 0, 25)
delayBox.Position = UDim2.new(0, 10, 0, 110)
delayBox.PlaceholderText = "Delay entre ataques (atual: 0.1)"
delayBox.Text = ""
delayBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
delayBox.TextColor3 = Color3.new(1, 1, 1)
delayBox.FocusLost:Connect(function()
    local val = tonumber(delayBox.Text)
    if val then
        getgenv().settings.delay = val
    end
end)

local moveBox = Instance.new("TextBox", frame)
moveBox.Size = UDim2.new(1, -20, 0, 25)
moveBox.Position = UDim2.new(0, 10, 0, 140)
moveBox.PlaceholderText = "Distância do avanço (padrão: 50)"
moveBox.Text = ""
moveBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
moveBox.TextColor3 = Color3.new(1, 1, 1)
moveBox.FocusLost:Connect(function()
    local val = tonumber(moveBox.Text)
    if val then
        getgenv().settings.move_distance = val
    end
end)

local flyBtn = Instance.new("TextButton", frame)
flyBtn.Size = UDim2.new(1, -20, 0, 30)
flyBtn.Position = UDim2.new(0, 10, 0, 170)
flyBtn.Text = "Voar pra frente"
flyBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
flyBtn.TextColor3 = Color3.new(1, 1, 1)
flyBtn.MouseButton1Click:Connect(moveForward)

local forceBtn = Instance.new("TextButton", frame)
forceBtn.Size = UDim2.new(1, -20, 0, 30)
forceBtn.Position = UDim2.new(0, 10, 0, 210)
forceBtn.Text = "Forçar avanço"
forceBtn.BackgroundColor3 = Color3.fromRGB(100, 60, 60)
forceBtn.TextColor3 = Color3.new(1, 1, 1)
forceBtn.MouseButton1Click:Connect(moveForward)

local minimize = Instance.new("TextButton", frame)
minimize.Size = UDim2.new(0, 30, 0, 30)
minimize.Position = UDim2.new(1, -35, 0, 0)
minimize.Text = "-"
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

local minimized = false
minimize.MouseButton1Click:Connect(function()
    minimized = not minimized
    for _, v in pairs(frame:GetChildren()) do
        if v ~= title and v ~= minimize then
            v.Visible = not minimized
        end
    end
    frame.Size = minimized and UDim2.new(0, 250, 0, 30) or UDim2.new(0, 250, 0, 260)
end)
