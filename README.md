-- LocalScript
-- Coloque em StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- 1. Referência aos RemoteEvents (Você precisa criá-los no ReplicatedStorage)
-- Se não existirem, o script vai esperar para sempre.
local remoteEvents = ReplicatedStorage:WaitForChild("LudoRemotes")
local rollDiceEvent = remoteEvents:WaitForChild("RollDice")
local movePieceEvent = remoteEvents:WaitForChild("MovePiece")

-- Criação da GUI
local gui = Instance.new("ScreenGui")
gui.Name = "LudoTestMenu"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(220, 250)
frame.Position = UDim2.new(0, 20, 0.5, -125)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
frame.BorderSizePixel = 0
frame.Parent = gui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundTransparency = 1
title.Text = "LUDO TEST MENU"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = frame

local function criarBotao(texto, y)
	local botao = Instance.new("TextButton")
	botao.Size = UDim2.new(1, -20, 0, 45)
	botao.Position = UDim2.fromOffset(10, y)
	botao.BackgroundColor3 = Color3.fromRGB(55, 110, 230)
	botao.TextColor3 = Color3.new(1, 1, 1)
	botao.TextSize = 16
	botao.Font = Enum.Font.GothamBold
	botao.Text = texto
	botao.Parent = frame
	return botao
end

local autoRoll = false
local autoMove = false

local rollButton = criarBotao("AUTO ROLL: OFF", 55)
local moveButton = criarBotao("AUTO MOVE: OFF", 110)
local testButton = criarBotao("TESTAR JOGADA", 165)

-- 2. Lógica do Botão de Rolar
rollButton.Activated:Connect(function()
	autoRoll = not autoRoll
	rollButton.Text = "AUTO ROLL: " .. (autoRoll and "ON" or "OFF")

	if autoRoll then
		-- Aqui você pode colocar um loop (task.spawn) para rolar automaticamente
		-- Por enquanto, apenas simula o clique uma vez:
		print("Solicitando rolagem de dado ao servidor...")
		rollDiceEvent:FireServer() -- Envia o comando para o servidor rolar o dado
	end
end)

-- 3. Lógica do Botão de Mover
moveButton.Activated:Connect(function()
	autoMove = not autoMove
	moveButton.Text = "AUTO MOVE: " .. (autoMove and "ON" or "OFF")
	
	if autoMove then
		print("Solicitando movimento automático de peça...")
		-- O ideal é que o servidor escolha a melhor peça baseada no dado atual
		-- Aqui mandamos um comando genérico para o servidor mover a peça
		movePieceEvent:FireServer() 
	end
end)

-- 4. Botão de Teste (Isso é ótimo para debug)
testButton.Activated:Connect(function()
	print("TESTE MANUAL: Rolar dado e mover peça imediatamente.")
	rollDiceEvent:FireServer()
	task.wait(1) -- Espera 1 segundo para o dado "rolar"
	movePieceEvent:FireServer()
end)
