--[[
    🎹 PIANO TILES - AUTO PLAY
    Local: StarterGui (dentro de um ScreenGui seu)
    Tipo: LocalScript
    
    Como usar:
    1. Crie um ScreenGui no StarterGui
    2. Insira este LocalScript dentro dele
    3. Ajuste AUTO_PLAY / CLICK_DELAY abaixo
]]

-- =========================
-- SERVIÇOS
-- =========================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local gui = script.Parent

-- =========================
-- CONFIGURAÇÃO
-- =========================
local AUTO_PLAY   = false
local CLICK_DELAY = 0.03   -- Delay para notas curtas
local HOLD_TICK   = 0.02   -- Frequência de checagem em notas longas

-- =========================
-- MENU
-- =========================
local menu = Instance.new("Frame")
menu.Name = "PianoModMenu"
menu.Size = UDim2.fromOffset(230, 130)
menu.Position = UDim2.new(0, 20, 0.5, -65)
menu.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
menu.BorderSizePixel = 0
menu.Active = true
menu.Draggable = true        -- permite arrastar o menu
menu.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = menu

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "🎹 Piano Tiles"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = menu

local button = Instance.new("TextButton")
button.Size = UDim2.new(1, -30, 0, 45)
button.Position = UDim2.fromOffset(15, 55)
button.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
button.Text = "AUTO PLAY: OFF"
button.TextColor3 = Color3.new(1, 1, 1)
button.TextSize = 16
button.Font = Enum.Font.GothamBold
button.AutoButtonColor = true
button.Parent = menu

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 8)
buttonCorner.Parent = button

-- Rastro/Status do botão
local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, 0, 0, 20)
status.Position = UDim2.fromOffset(0, 105)
status.BackgroundTransparency = 1
status.Text = "Aguardando notas..."
status.TextColor3 = Color3.fromRGB(180, 180, 180)
status.TextSize = 12
status.Font = Enum.Font.Gotham
status.Parent = menu

-- =========================
-- TOGGLE
-- =========================
button.MouseButton1Click:Connect(function()
    AUTO_PLAY = not AUTO_PLAY

    if AUTO_PLAY then
        button.Text = "AUTO PLAY: ON"
        button.BackgroundColor3 = Color3.fromRGB(40, 170, 80)
        status.Text = "Auto Play ativado ✅"
    else
        button.Text = "AUTO PLAY: OFF"
        button.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
        status.Text = "Auto Play desativado ❌"
    end
end)

-- =========================
-- TOCAR NOTA
-- =========================
local function tocarNota(nota)
    if not AUTO_PLAY then return end
    if not nota or not nota.Parent then return end
    if not nota:IsA("GuiObject") then return end

    local duracao = nota:GetAttribute("Duration") or 0

    -- Segurança: limita duração máxima para não travar
    if typeof(duracao) == "number" and duracao > 10 then
        duracao = 10
    end

    if duracao <= 0 then
        -- Nota curta
        nota:SetAttribute("Pressed", true)
        task.wait(CLICK_DELAY)

        if nota.Parent then
            nota:SetAttribute("Pressed", false)
        end
    else
        -- Nota longa: pressiona e mantém
        nota:SetAttribute("Pressed", true)

        local elapsed = 0
        while AUTO_PLAY and nota.Parent and elapsed < duracao do
            task.wait(HOLD_TICK)
            elapsed += HOLD_TICK
        end

        if nota.Parent then
            nota:SetAttribute("Pressed", false)
        end
    end
end

-- =========================
-- MONITOR DE NOTAS
-- =========================
local notasMonitoradas = {}

local function monitorarNota(obj)
    if not obj:IsA("GuiObject") then return end
    if obj:GetAttribute("PianoTile") ~= true then return end
    if notasMonitoradas[obj] then return end

    notasMonitoradas[obj] = true

    task.spawn(function()
        while obj.Parent do
            if AUTO_PLAY and not obj:GetAttribute("Played") then
                obj:SetAttribute("Played", true)

                tocarNota(obj)

                task.wait(0.02)

                if obj.Parent then
                    obj:SetAttribute("Played", false)
                end
            end

            task.wait(0.01)
        end

        notasMonitoradas[obj] = nil
    end)
end

-- =========================
-- INICIALIZAÇÃO
-- =========================
for _, obj in ipairs(gui:GetDescendants()) do
    monitorarNota(obj)
end

gui.DescendantAdded:Connect(monitorarNota)
