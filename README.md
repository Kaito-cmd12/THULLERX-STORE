-- ======================================================================
-- THULLERX STORE - CHALICE FINDER (ESPECIAL PARA SOLARA - PC)
-- ======================================================================

local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")

-- Limpa interface antiga para não duplicar
if CoreGui:FindFirstChild("ThullerxChalicePC") then
    CoreGui.ThullerxChalicePC:Destroy()
end

-- Interface Nativa Ultra-Rápida
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ThullerxChalicePC"
ScreenGui.Parent = CoreGui

local Frame = Instance.new("Frame")
Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
Frame.BorderColor3 = Color3.fromRGB(255, 50, 50)
Frame.BorderSizePixel = 2
Frame.Position = UDim2.new(0.02, 0, 0.25, 0)
Frame.Size = UDim2.new(0, 240, 0, 160)
Frame.Active = true
Frame.Draggable = true

local Title = Instance.new("TextLabel")
Title.Parent = Frame
Title.Size = UDim2.new(1, 0, 0, 32)
Title.BackgroundColor3 = Color3.fromRGB(28, 28, 28)
Title.Text = "THULLERX STORE [SOLARA]"
Title.TextColor3 = Color3.fromRGB(255, 50, 50)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 16

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Parent = Frame
StatusLabel.Position = UDim2.new(0, 0, 0.25, 0)
StatusLabel.Size = UDim2.new(1, 0, 0, 30)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "Status: Aguardando..."
StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.TextSize = 14

local CheckButton = Instance.new("TextButton")
CheckButton.Parent = Frame
CheckButton.Position = UDim2.new(0.08, 0, 0.5, 0)
CheckButton.Size = UDim2.new(0.84, 0, 0, 32)
CheckButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
CheckButton.Text = "Verificar Cálice"
CheckButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CheckButton.Font = Enum.Font.SourceSansBold
CheckButton.TextSize = 14

local AutoButton = Instance.new("TextButton")
AutoButton.Parent = Frame
AutoButton.Position = UDim2.new(0.08, 0, 0.75, 0)
AutoButton.Size = UDim2.new(0.84, 0, 0, 32)
AutoButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
AutoButton.Text = "Auto Checar: DESATIVADO"
AutoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
AutoButton.Font = Enum.Font.SourceSansBold
AutoButton.TextSize = 14

local AutoEnabled = false

-- Lógica de Detecção 100% Compatível com Solara
local function SafeCheckChalice()
    StatusLabel.Text = "Buscando no servidor..."
    StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
    
    task.wait(0.1)

    local found = false
    local holderName = ""

    -- Varredura segura apenas na raiz do workspace (evita travar a memória do Solara)
    for _, item in pairs(workspace:GetChildren()) do
        if item:IsA("Tool") or item:IsA("Model") then
            local n = string.lower(item.Name)
            if string.find(n, "chalice") or string.find(n, "god") then
                found = true
                holderName = "No Chão do Mapa"
                break
            end
        end
    end

    -- Varredura nos inventários dos jogadores
    if not found then
        for _, plr in pairs(Players:GetPlayers()) do
            local bp = plr:FindFirstChild("Backpack")
            local char = plr.Character

            if bp then
                for _, item in pairs(bp:GetChildren()) do
                    if string.find(string.lower(item.Name), "chalice") then
                        found = true
                        holderName = plr.DisplayName
                        break
                    end
                end
            end

            if not found and char then
                for _, item in pairs(char:GetChildren()) do
                    if item:IsA("Tool") and string.find(string.lower(item.Name), "chalice") then
                        found = true
                        holderName = plr.DisplayName
                        break
                    end
                end
            end

            if found then break end
        end
    end

    if found then
        StatusLabel.Text = "CÁLICE NO SERVIDO! (" .. holderName .. ")"
        StatusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
    else
        StatusLabel.Text = "Nenhum Cálice Detectado"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
    end
end

-- Conexão dos Botões
CheckButton.MouseButton1Click:Connect(function()
    SafeCheckChalice()
end)

AutoButton.MouseButton1Click:Connect(function()
    AutoEnabled = not AutoEnabled
    if AutoEnabled then
        AutoButton.Text = "Auto Checar: ATIVADO"
        AutoButton.BackgroundColor3 = Color3.fromRGB(40, 180, 40)
        SafeCheckChalice()
    else
        AutoButton.Text = "Auto Checar: DESATIVADO"
        AutoButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    end
end)

-- Loop em segundo plano a cada 10s (totalmente seguro para Solara)
task.spawn(function()
    while true do
        task.wait(10)
        if AutoEnabled then
            SafeCheckChalice()
        end
    end
end)
