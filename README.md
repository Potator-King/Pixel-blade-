-- Configurações
_G.AutoFarm = true -- Para parar, mude para false e execute novamente
local ferramentaNome = "Diamond Sword" 
local distancia = 4 -- Distância para ficar do Boss
-- Caminhos exatos baseados na sua imagem: workspace -> mobs -> BOSS / Boss2
local pastasAlvo = {"BOSS", "Boss2"} 

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Função para equipar a espada
local function equiparEspada()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("Humanoid") then return end
    
    local mochila = LocalPlayer.Backpack
    local espada = mochila:FindFirstChild(ferramentaNome)
    
    if espada then
        char.Humanoid:EquipTool(espada)
    end
end

-- Função para encontrar o inimigo vivo APENAS nas pastas BOSS e Boss2
local function getTarget()
    -- Verifica se a pasta "mobs" existe
    local pastaMobs = workspace:FindFirstChild("mobs")
    if not pastaMobs then return nil end

    for _, nomeSubPasta in pairs(pastasAlvo) do
        local pastaChefe = pastaMobs:FindFirstChild(nomeSubPasta)
        
        if pastaChefe then
            for _, npc in pairs(pastaChefe:GetChildren()) do
                -- Verifica se é um modelo válido, com vida e partes físicas
                if npc:IsA("Model") and npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
                    if npc.Humanoid.Health > 0 then
                        return npc -- Retorna o primeiro chefe vivo que achar
                    end
                end
            end
        end
    end
    return nil
end

local function farmar()
    while _G.AutoFarm do
        local target = getTarget()
        local char = LocalPlayer.Character
        
        if target and char and char:FindFirstChild("HumanoidRootPart") then
            local enemyRoot = target.HumanoidRootPart
            local enemyHumanoid = target.Humanoid
            
            -- Trava no alvo até ele morrer
            while _G.AutoFarm and enemyHumanoid.Health > 0 and char.Humanoid.Health > 0 do
                
                -- Segurança: se o boss sumir ou você morrer, para o ataque
                if not target:FindFirstChild("HumanoidRootPart") or not char:FindFirstChild("HumanoidRootPart") then
                    break
                end

                -- 1. Teleporte (Lock nas costas do Boss)
                char.HumanoidRootPart.CFrame = enemyRoot.CFrame * CFrame.new(0, 0, distancia)
                
                -- 2. Ataque
                equiparEspada()
                local espadaNaMao = char:FindFirstChild(ferramentaNome)
                if espadaNaMao then
                    espadaNaMao:Activate()
                end
                
                RunService.Heartbeat:Wait()
            end
        else
            -- Se não achar boss nas pastas BOSS/Boss2, espera um pouco
            task.wait(1)
        end
    end
end

farmar()
