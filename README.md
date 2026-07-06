--[[
    Cobalt Clone Hub
    Versión: 1.0
    Para ejecutar comandos de clonación
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

-- Variables de configuración
getgenv().CloneSettings = {
    Enabled = true,
    AutoClone = false,
    CloneInterval = 1,
    CurrentKey = "K",
    CurrentValue = 854848,
    ShowNotifications = true,
}

local Settings = getgenv().CloneSettings

-- Variables del hub
local Hub = {
    ToggleKey = Enum.KeyCode.RightShift,
    IsOpen = false,
    Dragging = false,
    DragInput = nil,
    DragStart = nil,
    StartPos = nil,
}

-- Funciones de clonación
local function ExecuteClone(value, key)
    if not Settings.Enabled then return end
    
    local keyToUse = key or Settings.CurrentKey
    local valueToUse = value or Settings.CurrentValue
    
    -- Ejecutar AttemptWithdrawCloneMachine
    local Event1 = ReplicatedStorage:FindFirstChild("Remote")
    if Event1 then
        local WithdrawEvent = Event1:FindFirstChild("AttemptWithdrawCloneMachine")
        if WithdrawEvent then
            WithdrawEvent:FireServer(keyToUse)
            if Settings.ShowNotifications then
                print("🔄 Clonación iniciada con clave: " .. keyToUse)
            end
        end
    end
    
    -- Ejecutar AttemptClone
    local Event2 = ReplicatedStorage:FindFirstChild("Remote")
    if Event2 then
        local CloneEvent = Event2:FindFirstChild("AttemptClone")
        if CloneEvent then
            CloneEvent:FireServer(valueToUse, keyToUse)
            if Settings.ShowNotifications then
                print("✅ Clon ejecutado con valor: " .. tostring(valueToUse) .. " y clave: " .. keyToUse)
            end
        end
    end
end

-- Sistema de auto-clonación
local AutoCloneRunning = false
local AutoCloneConnection = nil

local function StartAutoClone()
    if AutoCloneRunning then return end
    AutoCloneRunning = true
    
    AutoCloneConnection = game:GetService("RunService").Heartbeat:Connect(function()
        if Settings.AutoClone and Settings.Enabled then
            ExecuteClone()
        end
    end)
    
    print("🔄 Auto-clonación iniciada (Intervalo: " .. Settings.CloneInterval .. "s)")
end

local function StopAutoClone()
    AutoCloneRunning = false
    if AutoCloneConnection then
        AutoCloneConnection:Disconnect()
        AutoCloneConnection = nil
    end
    print("⏹️ Auto-clonación detenida")
end

-- Crear la interfaz del hub
local function CreateHub()
    -- ScreenGui principal
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "CobaltCloneHub"
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    ScreenGui.ResetOnSpawn = false

    -- Frame principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 400, 0, 480)
    MainFrame.Position = UDim2.new(0.5, -200, 0.5, -240)
    MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    MainFrame.BackgroundTransparency = 0.05
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Parent = ScreenGui

    -- Sombra y bordes
    local Shadow = Instance.new("Frame")
    Shadow.Name = "Shadow"
    Shadow.Size = UDim2.new(1, 0, 1, 0)
    Shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    Shadow.BackgroundTransparency = 0.7
    Shadow.BorderSizePixel = 0
    Shadow.Parent = MainFrame

    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 12)
    Corner.Parent = MainFrame

    local Stroke = Instance.new("UIStroke")
    Stroke.Color = Color3.fromRGB(80, 50, 150)
    Stroke.Thickness = 2
    Stroke.Parent = MainFrame

    -- Barra de título con gradiente
    local TitleBar = Instance.new("Frame")
    TitleBar.Name = "TitleBar"
    TitleBar.Size = UDim2.new(1, 0, 0, 45)
    TitleBar.BackgroundColor3 = Color3.fromRGB(40, 30, 60)
    TitleBar.BackgroundTransparency = 0.2
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame

    local TitleBarCorner = Instance.new("UICorner")
    TitleBarCorner.CornerRadius = UDim.new(0, 12)
    TitleBarCorner.Parent = TitleBar

    -- Icono y título
    local TitleText = Instance.new("TextLabel")
    TitleText.Name = "TitleText"
    TitleText.Size = UDim2.new(1, -60, 1, 0)
    TitleText.Position = UDim2.new(0, 10, 0, 0)
    TitleText.BackgroundTransparency = 1
    TitleText.Text = "⚡ Cobalt Clone Hub"
    TitleText.TextColor3 = Color3.fromRGB(180, 150, 255)
    TitleText.TextSize = 18
    TitleText.TextXAlignment = Enum.TextXAlignment.Left
    TitleText.Font = Enum.Font.GothamBold
    TitleText.Parent = TitleBar

    -- Botón de cerrar
    local CloseButton = Instance.new("TextButton")
    CloseButton.Name = "CloseButton"
    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -38, 0, 7)
    CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    CloseButton.BackgroundTransparency = 0.5
    CloseButton.BorderSizePixel = 0
    CloseButton.Text = "✕"
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.TextSize = 18
    CloseButton.Font = Enum.Font.GothamBold
    CloseButton.Parent = TitleBar

    local CloseCorner = Instance.new("UICorner")
    CloseCorner.CornerRadius = UDim.new(0, 6)
    CloseCorner.Parent = CloseButton

    CloseButton.MouseButton1Click:Connect(function()
        Hub.IsOpen = false
        MainFrame:TweenSize(UDim2.new(0, 0, 0, 0), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)
        task.wait(0.3)
        ScreenGui.Enabled = false
    end)

    -- Contenedor scroll
    local ScrollContainer = Instance.new("ScrollingFrame")
    ScrollContainer.Name = "ScrollContainer"
    ScrollContainer.Size = UDim2.new(1, -20, 1, -55)
    ScrollContainer.Position = UDim2.new(0, 10, 0, 50)
    ScrollContainer.BackgroundTransparency = 1
    ScrollContainer.BorderSizePixel = 0
    ScrollContainer.ScrollBarThickness = 4
    ScrollContainer.ScrollBarImageColor3 = Color3.fromRGB(80, 60, 120)
    ScrollContainer.Parent = MainFrame

    local Layout = Instance.new("UIListLayout")
    Layout.Padding = UDim.new(0, 8)
    Layout.SortOrder = Enum.SortOrder.LayoutOrder
    Layout.Parent = ScrollContainer

    -- Función para crear secciones
    local function CreateSection(text, icon)
        local Section = Instance.new("Frame")
        Section.Size = UDim2.new(1, 0, 0, 30)
        Section.BackgroundTransparency = 1
        Section.Parent = ScrollContainer

        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(1, 0, 1, 0)
        Label.BackgroundTransparency = 1
        Label.Text = (icon or "▸") .. " " .. text
        Label.TextColor3 = Color3.fromRGB(160, 140, 200)
        Label.TextSize = 14
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Font = Enum.Font.GothamSemibold
        Label.Parent = Section

        return Section
    end

    -- Función para crear toggle
    local function CreateToggle(text, settingKey, defaultValue)
        local ToggleFrame = Instance.new("Frame")
        ToggleFrame.Size = UDim2.new(1, 0, 0, 35)
        ToggleFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
        ToggleFrame.BackgroundTransparency = 0.3
        ToggleFrame.BorderSizePixel = 0
        ToggleFrame.Parent = ScrollContainer

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 6)
        Corner.Parent = ToggleFrame

        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(0.7, -10, 1, 0)
        Label.Position = UDim2.new(0, 10, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = text
        Label.TextColor3 = Color3.fromRGB(200, 200, 210)
        Label.TextSize = 13
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Font = Enum.Font.Gotham
        Label.Parent = ToggleFrame

        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(0, 50, 0, 25)
        Button.Position = UDim2.new(1, -60, 0, 5)
        Button.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        Button.BorderSizePixel = 0
        Button.Text = "OFF"
        Button.TextColor3 = Color3.fromRGB(255, 255, 255)
        Button.TextSize = 12
        Button.Font = Enum.Font.GothamBold
        Button.Parent = ToggleFrame

        local ButtonCorner = Instance.new("UICorner")
        ButtonCorner.CornerRadius = UDim.new(0, 4)
        ButtonCorner.Parent = Button

        -- Estado
        local state = Settings[settingKey] ~= nil and Settings[settingKey] or defaultValue
        Settings[settingKey] = state

        local function UpdateButton()
            if state then
                Button.BackgroundColor3 = Color3.fromRGB(100, 50, 200)
                Button.Text = "ON"
            else
                Button.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
                Button.Text = "OFF"
            end
        end
        UpdateButton()

        Button.MouseButton1Click:Connect(function()
            state = not state
            Settings[settingKey] = state
            UpdateButton()
            
            -- Manejar auto-clonación
            if settingKey == "AutoClone" then
                if state then
                    StartAutoClone()
                else
                    StopAutoClone()
                end
            end
        end)

        return Button
    end

    -- Función para crear slider
    local function CreateSlider(text, settingKey, min, max, defaultValue, format)
        local SliderFrame = Instance.new("Frame")
        SliderFrame.Size = UDim2.new(1, 0, 0, 50)
        SliderFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
        SliderFrame.BackgroundTransparency = 0.3
        SliderFrame.BorderSizePixel = 0
        SliderFrame.Parent = ScrollContainer

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 6)
        Corner.Parent = SliderFrame

        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(0.6, -10, 0, 20)
        Label.Position = UDim2.new(0, 10, 0, 2)
        Label.BackgroundTransparency = 1
        Label.Text = text
        Label.TextColor3 = Color3.fromRGB(200, 200, 210)
        Label.TextSize = 13
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Font = Enum.Font.Gotham
        Label.Parent = SliderFrame

        local ValueLabel = Instance.new("TextLabel")
        ValueLabel.Size = UDim2.new(0.3, 0, 0, 20)
        ValueLabel.Position = UDim2.new(0.7, 0, 0, 2)
        ValueLabel.BackgroundTransparency = 1
        ValueLabel.TextColor3 = Color3.fromRGB(160, 140, 200)
        ValueLabel.TextSize = 13
        ValueLabel.TextXAlignment = Enum.TextXAlignment.Right
        ValueLabel.Font = Enum.Font.Gotham
        ValueLabel.Parent = SliderFrame

        local Slider = Instance.new("Frame")
        Slider.Size = UDim2.new(0.9, -20, 0, 4)
        Slider.Position = UDim2.new(0, 10, 0, 35)
        Slider.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
        Slider.BorderSizePixel = 0
        Slider.Parent = SliderFrame

        local SliderCorner = Instance.new("UICorner")
        SliderCorner.CornerRadius = UDim.new(0, 2)
        SliderCorner.Parent = Slider

        local Fill = Instance.new("Frame")
        Fill.Size = UDim2.new(0.5, 0, 1, 0)
        Fill.BackgroundColor3 = Color3.fromRGB(120, 80, 220)
        Fill.BorderSizePixel = 0
        Fill.Parent = Slider

        local FillCorner = Instance.new("UICorner")
        FillCorner.CornerRadius = UDim.new(0, 2)
        FillCorner.Parent = Fill

        -- Estado
        local value = Settings[settingKey] ~= nil and Settings[settingKey] or defaultValue
        Settings[settingKey] = value

        local function UpdateSlider()
            local percent = (value - min) / (max - min)
            Fill.Size = UDim2.new(percent, 0, 1, 0)
            ValueLabel.Text = format and format(value) or tostring(value)
        end
        UpdateSlider()

        -- Input
        local function onSliderInput(input)
            local pos = input.Position.X
            local size = Slider.AbsoluteSize.X
            local percent = math.clamp((pos - Slider.AbsolutePosition.X) / size, 0, 1)
            value = min + (max - min) * percent
            value = math.round(value * 1000) / 1000
            Settings[settingKey] = value
            UpdateSlider()
        end

        Slider.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                onSliderInput(input)
                local connection
                connection = UserInputService.InputChanged:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseMovement then
                        onSliderInput(input)
                    end
                end)
                UserInputService.InputEnded:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        connection:Disconnect()
                    end
                end)
            end
        end)

        return Slider
    end

    -- Función para crear textbox
    local function CreateTextBox(text, settingKey, placeholder)
        local TextBoxFrame = Instance.new("Frame")
        TextBoxFrame.Size = UDim2.new(1, 0, 0, 35)
        TextBoxFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
        TextBoxFrame.BackgroundTransparency = 0.3
        TextBoxFrame.BorderSizePixel = 0
        TextBoxFrame.Parent = ScrollContainer

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 6)
        Corner.Parent = TextBoxFrame

        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(0.35, -10, 1, 0)
        Label.Position = UDim2.new(0, 10, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = text
        Label.TextColor3 = Color3.fromRGB(200, 200, 210)
        Label.TextSize = 13
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Font = Enum.Font.Gotham
        Label.Parent = TextBoxFrame

        local Input = Instance.new("TextBox")
        Input.Size = UDim2.new(0.55, -10, 0, 25)
        Input.Position = UDim2.new(0.45, 0, 0, 5)
        Input.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
        Input.BorderSizePixel = 0
        Input.TextColor3 = Color3.fromRGB(255, 255, 255)
        Input.TextSize = 12
        Input.Font = Enum.Font.Gotham
        Input.PlaceholderText = placeholder or "Valor..."
        Input.Parent = TextBoxFrame

        local InputCorner = Instance.new("UICorner")
        InputCorner.CornerRadius = UDim.new(0, 4)
        InputCorner.Parent = Input

        -- Cargar valor actual
        if Settings[settingKey] then
            Input.Text = tostring(Settings[settingKey])
        end

        Input.FocusLost:Connect(function()
            local val = Input.Text
            if val and val ~= "" then
                if settingKey == "CurrentKey" then
                    Settings[settingKey] = val
                    print("🔑 Clave actualizada a: " .. val)
                elseif settingKey == "CurrentValue" then
                    local numVal = tonumber(val)
                    if numVal then
                        Settings[settingKey] = numVal
                        print("🔢 Valor actualizado a: " .. numVal)
                    end
                end
            end
        end)

        return Input
    end

    -- Función para crear botón de acción
    local function CreateActionButton(text, color, action)
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(1, 0, 0, 40)
        Button.BackgroundColor3 = color or Color3.fromRGB(80, 50, 150)
        Button.BorderSizePixel = 0
        Button.Text = text
        Button.TextColor3 = Color3.fromRGB(255, 255, 255)
        Button.TextSize = 14
        Button.Font = Enum.Font.GothamBold
        Button.Parent = ScrollContainer

        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(0, 6)
        Corner.Parent = Button

        Button.MouseButton1Click:Connect(action)
        return Button
    end

    -- Construir la interfaz

    -- Sección: Control Principal
    CreateSection("🎮 Control Principal")
    CreateToggle("Módulo activado", "Enabled", true)
    CreateToggle("Auto-Clonación", "AutoClone", false)
    CreateToggle("Mostrar notificaciones", "ShowNotifications", true)
    CreateSlider("Intervalo de clonación", "CloneInterval", 0.1, 5, 1, function(v) return string.format("%.1fs", v) end)

    -- Sección: Configuración
    CreateSection("⚙️ Configuración")
    CreateTextBox("Clave (Key)", "CurrentKey", "Ej: K")
    CreateTextBox("Valor (Value)", "CurrentValue", "Ej: 854848")

    -- Sección: Acciones
    CreateSection("🚀 Acciones Rápidas")
    
    -- Botón para clonación manual
    CreateActionButton("⚡ Ejecutar Clonación", Color3.fromRGB(120, 80, 220), function()
        ExecuteClone()
        if Settings.ShowNotifications then
            print("⚡ Clonación ejecutada manualmente")
        end
    end)

    -- Botón para clonación con valores personalizados
    CreateActionButton("🔄 Clonación Rápida (K)", Color3.fromRGB(80, 50, 150), function()
        ExecuteClone(854848, "K")
    end)

    -- Botón para detener auto-clonación
    CreateActionButton("⏹️ Detener Auto-Clonación", Color3.fromRGB(200, 50, 50), function()
        Settings.AutoClone = false
        StopAutoClone()
        -- Actualizar toggle visualmente
        for _, child in pairs(ScrollContainer:GetChildren()) do
            if child:IsA("Frame") then
                for _, btn in pairs(child:GetChildren()) do
                    if btn:IsA("TextButton") and btn.Text == "ON" then
                        -- Buscar toggle de AutoClone
                        local label = child:FindFirstChild("TextLabel")
                        if label and label.Text == "Auto-Clonación" then
                            btn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
                            btn.Text = "OFF"
                        end
                    end
                end
            end
        end
    end)

    -- Sección: Información
    CreateSection("ℹ️ Información")
    
    local InfoLabel = Instance.new("TextLabel")
    InfoLabel.Size = UDim2.new(1, -20, 0, 80)
    InfoLabel.Position = UDim2.new(0, 10, 0, 0)
    InfoLabel.BackgroundTransparency = 1
    InfoLabel.Text = "Keybind: RightShift\nCreado con Cobalt\nhttps://github.com/notpoiu/cobalt"
    InfoLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
    InfoLabel.TextSize = 12
    InfoLabel.TextXAlignment = Enum.TextXAlignment.Left
    InfoLabel.TextYAlignment = Enum.TextYAlignment.Top
    InfoLabel.Font = Enum.Font.Gotham
    InfoLabel.Parent = ScrollContainer

    -- Aplicar layout
    Layout:Apply()

    -- Hacer la ventana arrastrable
    local function MakeDraggable()
        local dragging = false
        local dragStart = nil
        local startPos = nil

        TitleBar.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                dragStart = input.Position
                startPos = MainFrame.Position
            end
        end)

        TitleBar.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
                local delta = input.Position - dragStart
                MainFrame.Position = UDim2.new(
                    startPos.X.Scale,
                    startPos.X.Offset + delta.X,
                    startPos.Y.Scale,
                    startPos.Y.Offset + delta.Y
                )
            end
        end)
