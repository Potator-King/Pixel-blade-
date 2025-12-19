-- CONFIGURAÇÃO
_G.AutoFarm = true
local gloveName = "Bowling Glove"
local pastasAlvo = {"BOSS", "Boss2"}
local distancia = 1
local attackDelay = 0.08 -- NÃO diminua muito

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- BUSCAR BOSS
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

-- EQUIPAR LUVA
local function equiparLuva()
    local char = LocalPlayer.Character
    if not char then return nil end

    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return nil end

    local tool = char:FindFirstChild(gloveName)
        or LocalPlayer.Backpack:FindFirstChild(gloveName)

    if tool then
        humanoid:EquipTool(tool)
        return tool
    end
end

-- FARM
task.spawn(function()
    while _G.AutoFarm do
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then
            task.wait(1)
            continue
        end

        local target = getTarget()
        if not target then
            task.wait(1)
            continue
        end

        local humanoidEnemy = target:FindFirstChild("Humanoid")
        local enemyRoot = target:FindFirstChild("HumanoidRootPart")
        if not humanoidEnemy or not enemyRoot then
            task.wait(0.5)
            continue
        end

        local tool = equiparLuva()

        while _G.AutoFarm
        and humanoidEnemy.Health > 0
        and char:FindFirstChild("HumanoidRootPart")
        and char:FindFirstChild("Humanoid")
        and char.Humanoid.Health > 0 do

            -- Cola no boss
            char.HumanoidRootPart.CFrame =
                enemyRoot.CFrame * CFrame.new(0, 0, -distancia)

            -- Ataque CONTROLADO (DPS real)
            if tool then
                tool:Activate()
            end

            task.wait(attackDelay)
        end
    end
end)
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
