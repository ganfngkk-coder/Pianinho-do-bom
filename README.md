-- Piano Tiles Auto Player (corrigido)
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

local AUTO = false

-- ... (painel igual ao seu) ...

-- Detecta tiles: aceita GuiButton OU Frame com tamanho grande
local function isTile(obj)
    if not (obj:IsA("GuiButton") or obj:IsA("Frame")) then return false end
    if not obj.Visible then return false end
    local s = obj.AbsoluteSize
    if s.X < 20 or s.Y < 20 then return false end
    -- Aceita escuro OU qualquer coisa que pareça tile (ajuste aqui)
    local c = obj.BackgroundColor3
    return (c.R + c.G + c.B) / 3 < 0.35
end

local function fireMouse(tile, state)
    local pos = tile.AbsolutePosition + tile.AbsoluteSize / 2
    local inp = {
        UserInputType = Enum.UserInputType.MouseButton1,
        UserInputState = state,
        Position = Vector3.new(pos.X, pos.Y, 0),
    }
    -- Dispara no tile e em descendentes
    for _, o in ipairs({tile, table.unpack(tile:GetDescendants())}) do
        if o:IsA("GuiObject") then
            for _, evName in ipairs({"InputBegan", "InputEnded", "MouseButton1Down", "MouseButton1Up"}) do
                local ev = o[evName]
                if ev then pcall(function() ev:Fire(inp) end) end
            end
        end
    end
end

local function playTile(tile)
    if not tile.Parent or not tile.Visible then return end

    fireMouse(tile, Enum.UserInputState.Begin)

    local hold = math.clamp(tile.AbsoluteSize.Y / 180, 0.08, 3)
    if hold > 0.2 then
        task.wait(hold)
    end

    fireMouse(tile, Enum.UserInputState.End)
end

task.spawn(function()
    while task.wait(0.03) do
        if AUTO then
            for _, obj in ipairs(PlayerGui:GetDescendants()) do
                if isTile(obj) then
                    local t = obj:GetAttribute("LastAutoHit")
                    if not t or tick() - t > 0.3 then
                        obj:SetAttribute("LastAutoHit", tick())
                        task.spawn(playTile, obj)
                    end
                end
            end
        end
    end
end)
