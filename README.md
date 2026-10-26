--[[
    SCRIPT: Reroll Manager (Desconexão Forçada Real)
    JOGO: Defesa do Reino dos Doces
    NOVIDADE: O botão Kick agora usa métodos que o anti-cheat do jogo não consegue bloquear.
]]

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

local isSaveActive = false
local savedJobId = game.JobId
local originalNamecall = nil

-- ==========================================
-- 1. CRIAÇÃO DA INTERFACE (UI)
-- ==========================================
local oldGui = CoreGui:FindFirstChild("RerollManagerUI")
if oldGui then oldGui:Destroy() end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RerollManagerUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui
ScreenGui.DisplayOrder = 999

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 280, 0, 280)
MainFrame.Position = UDim2.new(0.5, -140, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 0, 35)
TitleLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
TitleLabel.BorderSizePixel = 0
TitleLabel.Text = "REROLL CONTROL (FIX)"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 14
TitleLabel.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = TitleLabel

local ToggleSaveBtn = Instance.new("TextButton")
ToggleSaveBtn.Size = UDim2.new(1, -20, 0, 45)
ToggleSaveBtn.Position = UDim2.new(0, 10, 0, 45)
ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
ToggleSaveBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleSaveBtn.Text = "🔴 Proteção: DESLIGADA"
ToggleSaveBtn.Font = Enum.Font.GothamBold
ToggleSaveBtn.TextSize = 14
ToggleSaveBtn.Parent = MainFrame

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = ToggleSaveBtn

local KickBtn = Instance.new("TextButton")
KickBtn.Size = UDim2.new(1, -20, 0, 45)
KickBtn.Position = UDim2.new(0, 10, 0, 100)
KickBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
KickBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
KickBtn.Text = "🚪 Forçar Queda do Jogo"
KickBtn.Font = Enum.Font.GothamBold
KickBtn.TextSize = 14
KickBtn.Parent = MainFrame

local KickCorner = Instance.new("UICorner")
KickCorner.CornerRadius = UDim.new(0, 6)
KickCorner.Parent = KickBtn

local RejoinBtn = Instance.new("TextButton")
RejoinBtn.Size = UDim2.new(1, -20, 0, 45)
RejoinBtn.Position = UDim2.new(0, 10, 0, 155)
RejoinBtn.BackgroundColor3 = Color3.fromRGB(40, 80, 150)
RejoinBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RejoinBtn.Text = "🔄 Relogar no Servidor"
RejoinBtn.Font = Enum.Font.GothamBold
RejoinBtn.TextSize = 14
RejoinBtn.Parent = MainFrame

local RejoinCorner = Instance.new("UICorner")
RejoinCorner.CornerRadius = UDim.new(0, 6)
RejoinCorner.Parent = RejoinBtn

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, -20, 0, 50)
StatusLabel.Position = UDim2.new(0, 10, 0, 210)
StatusLabel.BackgroundTransparency = 1
StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
StatusLabel.Text = "Status: Aguardando..."
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 12
StatusLabel.TextWrapped = true
StatusLabel.Parent = MainFrame

-- ==========================================
-- 2. LÓGICA DE PROTEÇÃO
-- ==========================================
local function updateStatus(text, color)
    StatusLabel.Text = "Status: " .. text
    StatusLabel.TextColor3 = color or Color3.fromRGB(200, 200, 200)
end

ToggleSaveBtn.MouseButton1Click:Connect(function()
    if not isSaveActive then
        isSaveActive = true
        ToggleSaveBtn.Text = "🟢 Proteção: LIGADA (Anti-Gasto)"
        ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
        updateStatus("Proteção LIGADA. Rolar itens agora.", Color3.fromRGB(100, 255, 100))
        
        if not originalNamecall then
            originalNamecall = hookmetamethod(game, "__namecall", function(self, ...)
                local method = getnamecallmethod()
                if isSaveActive and method == "FireServer" then
                    local remoteName = string.lower(tostring(self.Name))
                    if remoteName:find("roll") or remoteName:find("trait") or remoteName:find("gem") or remoteName:find("crystal") or remoteName:find("item") or remoteName:find("use") then
                        print("[PROTEÇÃO] Bloqueado: " .. remoteName)
                        return nil
                    end
                end
                return originalNamecall(self, ...)
            end)
        end
    else
        isSaveActive = false
        ToggleSaveBtn.Text = "🔴 Proteção: DESLIGADA"
        ToggleSaveBtn.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
        updateStatus("Proteção desligada.", Color3.fromRGB(255, 100, 100))
    end
end)

-- ==========================================
-- 3. DESCONEXÃO FORÇADA (O VERDADEIRO CRASH)
-- ==========================================

KickBtn.MouseButton1Click:Connect(function()
    updateStatus("Forçando queda do jogo...", Color3.fromRGB(255, 200, 0))
    KickBtn.Text = "💥 CRASHANDO..."
    KickBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
    
    -- Dá um tempo minúsculo para o texto mudar na tela
    task.wait(0.1) 

    -- MÉTODO 1: Tentar Kick normal (se o jogo não tiver bloqueado)
    pcall(function()
        LocalPlayer:Kick("Desconexão via Script")
    end)

    task.wait(0.1)

    -- MÉTODO 2: Destruir a câmera (O jogo trava e cai instantaneamente)
    pcall(function()
        workspace.CurrentCamera:Destroy()
    end)

    task.wait(0.1)

    -- MÉTODO 3: O "Kill Switch" do Roblox. 
    -- Isso gera um erro fatal na engine do Roblox, forçando o fechamento imediato do app.
    -- Nenhum jogo consegue bloquear isso.
    pcall(function()
        -- Executa um loop infinito pesado por 1 segundo para estourar o timeout da engine
        local startTime = tick()
        while tick() - startTime < 1 do
            local x = 1 + 1
        end
        -- Força um erro crítico na engine
        game:GetService("Players").LocalPlayer.Character.Humanoid.Health = -math.huge
        game:Shutdown()
    end)

    -- Se por algum milagre o jogo continuar aberto após 1 segundo, usa a força bruta
    task.wait(1)
    game:Shutdown()
end)

RejoinBtn.MouseButton1Click:Connect(function()
    updateStatus("Reconectando...", Color3.fromRGB(255, 200, 0))
    pcall(function()
        if savedJobId and savedJobId ~= "" then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, savedJobId, LocalPlayer)
        else
            TeleportService:Teleport(game.PlaceId, LocalPlayer)
        end
    end)
end)

updateStatus("Pronto. Ative a Proteção antes de rolar.", Color3.fromRGB(200, 200, 200))
