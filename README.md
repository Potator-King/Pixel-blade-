-- Configurações
_G.AutoFarm = true -- Para parar, mude para false e execute novamente
local ferramentaNome = "Diamond Sword" -- Nome da espada
local distancia = 4 -- Distância do inimigo
local pastasAlvo = {"Boss", "Boss2"} -- Nomes exatos das pastas que você mencionou

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Função para equipar a espada
local function equiparEspada()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("Humanoid") then return end
    
    local mochila = LocalPlayer.Backpack
    local espada = mochila:FindFirstChild(ferramentaNome)
    
    -- Se a espada estiver na mochila, equipa
    if espada then
        char.Humanoid:EquipTool(espada)
    end
end

-- Função para encontrar o inimigo vivo mais próximo dentro das pastas
local function getTarget()
    for _, nomePasta in pairs(pastasAlvo) do
        local pasta = workspace:FindFirstChild(nomePasta)
        if pasta then
            for _, npc in pairs(pasta:GetChildren()) do
                -- Verifica se é um modelo válido, com vida e partes físicas
                if npc:IsA("Model") and npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
                    if npc.Humanoid.Health > 0 then
                        return npc -- Retorna o primeiro inimigo vivo que achar
                    end
                end
            end
        end
    end
    return nil -- Retorna nulo se não achar ninguém vivo
end

local function farmar()
    while _G.AutoFarm do
        local target = getTarget()
        local char = LocalPlayer.Character
        
        if target and char and char:FindFirstChild("HumanoidRootPart") then
            local enemyRoot = target.HumanoidRootPart
            local enemyHumanoid = target.Humanoid
            
            -- Loop de combate FOCADO: Só sai daqui quando esse boss específico morrer
            while _G.AutoFarm and enemyHumanoid.Health > 0 and char.Humanoid.Health > 0 do
                
                -- Se o inimigo ou o player perderem as partes físicas, para o loop
                if not target:FindFirstChild("HumanoidRootPart") or not char:FindFirstChild("HumanoidRootPart") then
                    break
                end

                -- 1. Teleporte (Lock)
                -- Fica atrás do inimigo. O CFrame mantém a rotação correta.
                char.HumanoidRootPart.CFrame = enemyRoot.CFrame * CFrame.new(0, 0, distancia)
                
                -- 2. Ataque
                equiparEspada()
                local espadaNaMao = char:FindFirstChild(ferramentaNome)
                if espadaNaMao then
                    espadaNaMao:Activate()
                end
                
                -- Velocidade do loop (Heartbeat é frame a frame)
                RunService.Heartbeat:Wait()
            end
        else
            -- Se não tem ninguém vivo nas pastas, espera 1 segundo antes de procurar de novo
            task.wait(1)
        end
    end
end

farmar()
