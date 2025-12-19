-- Configurações
_G.AutoFarm = true -- Mude para false para parar
local ferramentaNome = "Diamond Sword" -- Nome exato que vi na sua print
local distancia = 5 -- Distância para ficar do Boss (evita bugar dentro dele)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Função para equipar a espada
local function equiparEspada()
    local char = LocalPlayer.Character
    if not char then return end
    
    local mochila = LocalPlayer.Backpack
    local espada = mochila:FindFirstChild(ferramentaNome)
    
    -- Se a espada estiver na mochila, equipa
    if espada then
        char.Humanoid:EquipTool(espada)
    end
end

-- Função principal de ataque
local function farmar()
    while _G.AutoFarm do
        local target = nil
        local char = LocalPlayer.Character
        
        -- Procura um inimigo vivo na Workspace
        -- Baseado nas suas prints, os bosses estão soltos na Workspace
        for _, v in pairs(workspace:GetChildren()) do
            -- Verifica se é um modelo, se tem vida e não é você mesmo
            if v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Name ~= LocalPlayer.Name then
                if v.Humanoid.Health > 0 then
                    target = v
                    break -- Foca neste alvo até morrer ou mudar
                end
            end
        end

        -- Se achou um alvo
        if target and char and char:FindFirstChild("HumanoidRootPart") then
            local enemyRoot = target.HumanoidRootPart
            local enemyHumanoid = target.Humanoid
            
            -- Loop de combate: Enquanto o inimigo tiver vida e o AutoFarm estiver ligado
            while _G.AutoFarm and enemyHumanoid.Health > 0 and char.Humanoid.Health > 0 do
                
                -- 1. Teleporta para trás do inimigo (CFrame)
                -- O RenderStepped garante que você grude nele mesmo se ele andar
                char.HumanoidRootPart.CFrame = enemyRoot.CFrame * CFrame.new(0, 0, distancia)
                
                -- 2. Equipa a espada se não estiver equipada
                equiparEspada()
                
                -- 3. Ativa a espada (Clica)
                local espadaNaMao = char:FindFirstChild(ferramentaNome)
                if espadaNaMao then
                    espadaNaMao:Activate()
                end
                
                -- Evita crashar o jogo (espera um frame minúsculo)
                RunService.Heartbeat:Wait()
            end
        else
            -- Se não tem inimigos, espera um pouco antes de procurar de novo
            task.wait(1)
        end
        task.wait()
    end
end

-- Inicia o script
farmar()
