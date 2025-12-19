-- CONFIGURAÇÃO --------------------------
_G.AutoFarm = true          -- liga/desliga o farm
local gloveName = "Bowling Glove" -- nome exato da sua luva
local pastasAlvo = {"BOSS","Boss2"} -- pastas onde procurar boss
local distancia = 1.0        -- quão perto do HRP do boss (ajusta pra 0.8-1.2)
local spamDelay = 0.03      -- delay entre ciclos de spam (menor = mais rápido)
local spamPerCycle = 8      -- quantas ativações por ciclo
local reEquipEvery = 6      -- a cada N ciclos tenta re-equip (ajuda "resetar" cooldown client-side)
local useFireTouch = true   -- tenta usar firetouchinterest se disponível
------------------------------------------

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- helper: procura boss vivo nas pastas definidas
local function getTarget()
    local mobs = workspace:FindFirstChild("mobs")
    if not mobs then return nil end
    for _, pastaName in ipairs(pastasAlvo) do
        local pasta = mobs:FindFirstChild(pastaName)
        if pasta then
            for _, npc in ipairs(pasta:GetChildren()) do
                if npc:IsA("Model")
                and npc:FindFirstChild("Humanoid")
                and npc:FindFirstChild("HumanoidRootPart")
                and npc.Humanoid.Health > 0 then
                    return npc
                end
            end
        end
    end
    return nil
end

-- equipa uma tool pelo nome (backpack -> humanoid)
local function equipToolByName(name)
    local char = LocalPlayer.Character
    if not char then return nil end
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return nil end

    local tool = char:FindFirstChild(name) or LocalPlayer.Backpack:FindFirstChild(name)
    if tool then
        -- usa EquipTool para garantir
        pcall(function() humanoid:EquipTool(tool) end)
        return tool
    end
    return nil
end

-- força re-equip para tentar "resetar" cooldown client-side
local function forceReequip(tool)
    local char = LocalPlayer.Character
    if not char then return end
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end
    pcall(function()
        humanoid:UnequipTools()
        task.wait(0.04)
        if tool.Parent == LocalPlayer.Backpack or tool.Parent == char then
            humanoid:EquipTool(tool)
        else
            -- tenta encontrar novamente
            local alt = char:FindFirstChild(tool.Name) or LocalPlayer.Backpack:FindFirstChild(tool.Name)
            if alt then humanoid:EquipTool(alt) end
        end
    end)
end

-- tenta simular toque (se o jogo usar Touched)
local function tryFireTouch(tool, target)
    if not useFireTouch then return end
    if not tool or not tool:FindFirstChild("Handle") or not target then return end
    local handle = tool:FindFirstChild("Handle")
    -- percorre as partes do boss e firetouchinterest
    pcall(function()
        for _, part in ipairs(target:GetDescendants()) do
            if part:IsA("BasePart") then
                -- firetouchinterest(handle, part, 0) / (1)
                pcall(function() firetouchinterest(handle, part, 0) end)
                pcall(function() firetouchinterest(handle, part, 1) end)
            end
        end
    end)
end

-- função principal de spam da luva (tenta várias técnicas)
local function spamGloveOnTarget(target)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end

    -- equipa a luva
    local tool = equipToolByName(gloveName)
    if not tool then return end

    -- cola no boss (perto do centro do HRP)
    local enemyRoot = target:FindFirstChild("HumanoidRootPart")
    if not enemyRoot then return end
    char.HumanoidRootPart.CFrame = enemyRoot.CFrame * CFrame.new(0, 0, -distancia)

    -- loop de ataque: spam + tentativas de re-equip + firetouch
    local cycle = 0
    while _G.AutoFarm
    and target
    and target.Parent
    and target:FindFirstChild("Humanoid")
    and target.Humanoid.Health > 0
    and char
    and char:FindFirstChild("HumanoidRootPart")
    and char:FindFirstChild("Humanoid")
    and char.Humanoid.Health > 0 do

        cycle = cycle + 1

        -- 1) tenta ativar várias vezes (Activate)
        for i = 1, spamPerCycle do
            pcall(function()
                -- método com colon
                if tool.Parent == char then
                    tool:Activate()
                else
                    -- tenta equipar se não estiver equipado
                    tool = equipToolByName(gloveName) or tool
                    if tool and tool.Parent == char then
                        tool:Activate()
                    end
                end
            end)
        end

        -- 2) tenta firetouchinterest (se possível)
        pcall(function() tryFireTouch(tool, target) end)

        -- 3) de vez em quando força re-equip pra "resetar" cooldown client-side
        if cycle % reEquipEvery == 0 then
            pcall(function() forceReequip(tool) end)
        end

        -- 4) mantém posição colada no boss (em caso de knockback)
        pcall(function()
            if enemyRoot and char and char:FindFirstChild("HumanoidRootPart") then
                char.HumanoidRootPart.CFrame = enemyRoot.CFrame * CFrame.new(0, 0, -distancia)
            end
        end)

        task.wait(spamDelay)
    end
end

-- loop principal de farm
local function farmar()
    while _G.AutoFarm do
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then
            task.wait(1)
            continue
        end

        local target = getTarget()
        if target and target.Parent then
            -- se encontrar target, chama a função de spam da luva
            spamGloveOnTarget(target)
        else
            task.wait(1)
        end
    end
end

-- inicia
task.spawn(farmar)
