-- ======================================================================
-- THULLERX STORE - HUB CORRIGIDO (CODES & DAMAGE FIX)
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Blox Fruits Quest Hub",
    TabWidth = 160,
    Size = UDim2.fromOffset(630, 480),
    Acrylic = true,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.K
})

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualUser = game:GetService("VirtualUser")
local TweenService = game:GetService("TweenService")

-- Referência direta aos Remotes confirmados no seu Explorer
local RemotesFolder = ReplicatedStorage:WaitForChild("Remotes")
local CommF_ = RemotesFolder:WaitForChild("CommF_")

_G.AutoFarmLevel = false
_G.TweenSpeed = 250
_G.FarmHeight = 3 -- Reduzido para 3 studs para garantir alcance da arma
_G.CurrentTween = nil

-- LISTA DE CÓDIGOS DA SUA IMAGEM
local PromoCodes = {
    "EASTEREXP", "fudd10", "fudd10_V2", "Chandler", "BIGNEWS", 
    "KITT_RESET", "Sub2UncleKizaru", "SUB2GAMERROBOT_RESET1", 
    "Sub2Fer999", "Enyu_is_Pro", "JCWK", "StarcodeHEO", "MagicBUS", 
    "KittGaming", "Sub2CaptainMaui", "Sub2OfficialNoobie", "TheGreatAce", 
    "Sub2NoobMaster123", "Sub2Daigrock", "Axiore", "StrawHatMaine", 
    "TantaiGaming", "Bluxxy", "SUB2GAMERROBOT_EXP1"
}

local function StopMovement()
    if _G.CurrentTween then
        _G.CurrentTween:Cancel()
        _G.CurrentTween = nil
    end
end

local function TweenTo(targetCFrame)
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local hrp = character.HumanoidRootPart
        local distance = (hrp.Position - targetCFrame.Position).Magnitude
        local duration = distance / _G.TweenSpeed
        
        StopMovement()
        local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear)
        _G.CurrentTween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
        _G.CurrentTween:Play()
        return _G.CurrentTween
    end
end

-- SISTEMA DE ATAQUE (Garante equipar + simular clique + registrar no remote)
local function ExecuteAttack()
    pcall(function()
        local char = LocalPlayer.Character
        if not char then return end
        
        -- 1. Equipa a primeira ferramenta do inventário se mão estiver vazia
        local tool = char:FindFirstChildOfClass("Tool")
        if not tool then
            for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
                if item:IsA("Tool") then
                    char.Humanoid:EquipTool(item)
                    break
                end
            end
        else
            -- 2. Dispara a animação/ativação local
            tool:Activate()
            
            -- 3. Dispara o Remote de ataque oficial do Blox Fruits
            CommF_:InvokeServer("RegisterAttack")
            
            -- 4. Simula o clique esquerdo do mouse no motor do jogo
            VirtualUser:CaptureController()
            VirtualUser:Button1Down(Vector2.new(1e4, 1e4))
            VirtualUser:Button1Up(Vector2.new(1e4, 1e4))
        end
    end)
end

-- INTERFACE
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "sword" }),
    Movement = Window:AddTab({ Title = "Ajustes", Icon = "move" })
}

Tabs.Main:AddSection("Resgate de Códigos")

Tabs.Main:AddButton({
    Title = "Resgatar Todos os Códigos (Foto)",
    Description = "Executa a lista enviada via CommF_",
    Callback = function()
        Fluent:Notify({ Title = "THULLERX STORE", Content = "Iniciando resgate dos códigos...", Duration = 3 })
        
        task.spawn(function()
            for _, code in ipairs(PromoCodes) do
                pcall(function()
                    CommF_:InvokeServer("RedeemCode", tostring(code))
                end)
                task.wait(0.4) -- Delay de segurança para o servidor registrar
            end
            
            Fluent:Notify({ Title = "THULLERX STORE", Content = "Processo de códigos finalizado!", Duration = 4 })
        end)
    end
})

Tabs.Main:AddSection("Farm de NPC")

Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Farm / Ataque NPC",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then StopMovement() end
    end
})

Tabs.Movement:AddSlider("HeightSlider", {
    Title = "Distância do NPC (Altura)",
    Description = "Se o dano não registrar, reduza para 1 ou 2",
    Default = 3,
    Min = 0,
    Max = 8,
    Rounding = 0,
    Callback = function(V) _G.FarmHeight = V end
})

-- LOOP PRINCIPAL DE ATAQUE E POSICIONAMENTO
task.spawn(function()
    while true do
        task.wait(0.05)
        if _G.AutoFarmLevel then
            pcall(function()
                local enemies = workspace:FindFirstChild("Enemies")
                local targetEnemy = nil

                -- Encontra o NPC mais próximo no Workspace
                if enemies then
                    for _, enemy in pairs(enemies:GetChildren()) do
                        if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                            targetEnemy = enemy
                            break
                        end
                    end
                end

                -- Se encontrou o NPC, cola nele e ataca
                if targetEnemy then
                    local mobHrp = targetEnemy.HumanoidRootPart
                    local playerHrp = LocalPlayer.Character.HumanoidRootPart
                    
                    -- Posiciona exatamente em cima do mob
                    playerHrp.CFrame = mobHrp.CFrame * CFrame.new(0, _G.FarmHeight, 0)
                    
                    -- Ataca apenas se estiver bem perto
                    if (playerHrp.Position - mobHrp.Position).Magnitude <= 15 then
                        ExecuteAttack()
                    end
                end
            end)
        end
    end
end)

Window:SelectTab(1)
