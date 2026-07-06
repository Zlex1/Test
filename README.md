-- Script de SPAM para os códigos Cobalt (Clone Machine)
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local gui = player:WaitForChild("PlayerGui")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Referências aos eventos remotos
local AttemptWithdrawCloneMachine = ReplicatedStorage:FindFirstChild("Remote"):FindFirstChild("AttemptWithdrawCloneMachine")
local AttemptClone = ReplicatedStorage:FindFirstChild("Remote"):FindFirstChild("AttemptClone")

-- Criar interface
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = gui
screenGui.ResetOnSpawn = false

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 350, 0, 200)
frame.Position = UDim2.new(0.5, -175, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BackgroundTransparency = 0.1
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0.15, 0)
title.Text = "SPAM CLONE MACHINE"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.Parent = frame

-- Status
local statusText = Instance.new("TextLabel")
statusText.Size = UDim2.new(1, 0, 0.15, 0)
statusText.Position = UDim2.new(0, 0, 0.18, 0)
statusText.Text = "DESLIGADO"
statusText.TextColor3 = Color3.fromRGB(255, 0, 0)
statusText.BackgroundTransparency = 1
statusText.Font = Enum.Font.GothamBold
statusText.TextSize = 16
statusText.Parent = frame

-- Botão toggle
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0.4, 0, 0.2, 0)
toggleBtn.Position = UDim2.new(0.3, 0, 0.35, 0)
toggleBtn.Text = "ATIVAR"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 16
toggleBtn.Parent = frame

-- Configurações
local configFrame = Instance.new("Frame")
configFrame.Size = UDim2.new(1, 0, 0.35, 0)
configFrame.Position = UDim2.new(0, 0, 0.6, 0)
configFrame.BackgroundTransparency = 1
configFrame.Parent = frame

-- Delay
local delayLabel = Instance.new("TextLabel")
delayLabel.Size = UDim2.new(0.4, 0, 0.4, 0)
delayLabel.Text = "Delay: 0.1s"
delayLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
delayLabel.BackgroundTransparency = 1
delayLabel.Font = Enum.Font.Gotham
delayLabel.TextSize = 12
delayLabel.Parent = configFrame

local delaySlider = Instance.new("TextBox")
delaySlider.Size = UDim2.new(0.4, 0, 0.4, 0)
delaySlider.Position = UDim2.new(0.5, 0, 0, 0)
delaySlider.Text = "0.1"
delaySlider.TextColor3 = Color3.fromRGB(255, 255, 255)
delaySlider.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
delaySlider.Font = Enum.Font.Gotham
delaySlider.TextSize = 12
delaySlider.Parent = configFrame

-- Contador
local counterLabel = Instance.new("TextLabel")
counterLabel.Size = UDim2.new(1, 0, 0.3, 0)
counterLabel.Position = UDim2.new(0, 0, 0.5, 0)
counterLabel.Text = "Enviados: 0"
counterLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
counterLabel.BackgroundTransparency = 1
counterLabel.Font = Enum.Font.Gotham
counterLabel.TextSize = 12
counterLabel.Parent = configFrame

-- Variáveis de controle
local spamming = false
local connection = nil
local counter = 0
local delayTime = 0.1

-- Função para enviar os eventos
local function sendEvents()
    if AttemptWithdrawCloneMachine and AttemptClone then
        -- Evento 1: AttemptWithdrawCloneMachine com "K"
        AttemptWithdrawCloneMachine:FireServer("K")
        
        -- Evento 2: AttemptClone com 854848 e "K"
        AttemptClone:FireServer(854848, "K")
        
        counter = counter + 1
        counterLabel.Text = "Enviados: " .. counter
    else
        print("❌ Eventos remotos não encontrados!")
        toggleSpam() -- Desativa se não encontrar
    end
end

-- Função para ativar/desativar spam
local function toggleSpam()
    spamming = not spamming
    
    if spamming then
        statusText.Text = "ATIVADO"
        statusText.TextColor3 = Color3.fromRGB(0, 255, 0)
        toggleBtn.Text = "DESATIVAR"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
        
        -- Atualizar delay
        delayTime = tonumber(delaySlider.Text) or 0.1
        if delayTime < 0.01 then delayTime = 0.01 end
        
        print("✅ SPAM ATIVADO! Delay: " .. delayTime .. "s")
        
        -- Iniciar loop de spam
        connection = game:GetService("RunService").Heartbeat:Connect(function()
            sendEvents()
            task.wait(delayTime)
        end)
    else
        statusText.Text = "DESLIGADO"
        statusText.TextColor3 = Color3.fromRGB(255, 0, 0)
        toggleBtn.Text = "ATIVAR"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
        
        if connection then
            connection:Disconnect()
            connection = nil
        end
        
        print("⏹️ SPAM DESATIVADO")
    end
end

-- Conectar botão
toggleBtn.MouseButton1Click:Connect(toggleSpam)

-- Atualizar delay quando mudar
delaySlider.FocusLost:Connect(function()
    local newDelay = tonumber(delaySlider.Text)
    if newDelay then
        delayTime = math.max(0.01, newDelay)
        delayLabel.Text = "Delay: " .. delayTime .. "s"
        
        -- Se estiver ativo, reiniciar com novo delay
        if spamming then
            toggleSpam()
            task.wait(0.1)
            toggleSpam()
        end
    else
        delaySlider.Text = tostring(delayTime)
    end
end)

-- Tecla de atalho: Insert
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Insert then
        toggleSpam()
    end
end)

-- Verificar se os eventos existem
if not AttemptWithdrawCloneMachine or not AttemptClone then
    statusText.Text = "ERRO: Eventos não encontrados!"
    statusText.TextColor3 = Color3.fromRGB(255, 200, 0)
    toggleBtn.Visible = false
    print("❌ Eventos remotos não encontrados!")
    print("Verifique se os caminhos estão corretos:")
    print("- ReplicatedStorage.Remote.AttemptWithdrawCloneMachine")
    print("- ReplicatedStorage.Remote.AttemptClone")
else
    print("✅ Eventos encontrados!")
    print("✅ Script carregado com sucesso!")
    print("📌 Pressione INSERT ou clique no botão para ativar/desativar")
end

-- Reset ao morrer
game:GetService("Players").LocalPlayer.CharacterAdded:Connect(function()
    if spamming then
        toggleSpam()
    end
end)
