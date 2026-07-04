--[[
    ORA CORE - Com Preview 3D Sempre Ativo (SEM NOTIFICAÇÕES)
]]

local Luna = loadstring(game:HttpGet("https://raw.githubusercontent.com/Nebula-Softworks/Luna-Interface-Suite/refs/heads/master/source.lua", true))()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

-- ================= CONFIGURAÇÕES =================
local HubConfig = {
    Name = "Ora Core",
    Subtitle = "Hub de Customização",
    LogoID = "120245531583106",
    LoadingEnabled = true,
    LoadingTitle = "Ora Core",
    LoadingSubtitle = "Carregando...",
    ConfigSettings = {
        RootFolder = nil,
        ConfigFolder = "ORACORE"
    },
    KeySystem = false
}

-- ================= CRIAÇÃO DA JANELA =================
local Window = Luna:CreateWindow(HubConfig)

-- ================= VARIÁVEIS DO PREVIEW 3D =================
local previewCamera = nil
local previewModel = nil
local isPreviewActive = false
local previewScreenGui = nil
local viewportFrame = nil
local rotationAngle = 0
local previewCreated = false

-- ================= FUNÇÕES PRINCIPAIS =================

local function sendColor(cmd, r, g, b)
    if not LP:FindFirstChild("startevent") then 
        return false 
    end
    
    local success, err = pcall(function()
        LP.startevent:FireServer(
            cmd,
            string.format("%.2f,%.2f,%.2f", r/255, g/255, b/255)
        )
    end)
    
    if isPreviewActive then
        updatePreview()
    end
    
    return success
end

local function applyMask(mask)
    if LP:FindFirstChild("startevent") then
        local success, err = pcall(function()
            LP.startevent:FireServer("mask", mask)
        end)
        
        if isPreviewActive and success then
            updatePreview()
        end
        
        return success
    end
    return false
end

local function applyCustomOutfit(shirtId, pantsId)
    if LP:FindFirstChild("startevent") then
        local success, err = pcall(function()
            LP.startevent:FireServer("customoutfit", shirtId, pantsId)
        end)
        
        if isPreviewActive and success then
            updatePreview()
        end
        
        return success
    end
    return false
end

local function resetOutfit()
    if LP:FindFirstChild("startevent") then
        local success, err = pcall(function()
            LP.startevent:FireServer("resetoutfit")
        end)
        
        if isPreviewActive and success then
            updatePreview()
        end
        
        return success
    end
    return false
end

-- ================= FUNÇÕES DO PREVIEW 3D =================

local function createPreview()
    if not LP.Character or not LP.Character:FindFirstChild("Humanoid") then
        return false
    end
    
    -- Remove preview antigo
    if previewModel then
        previewModel:Destroy()
        previewModel = nil
    end
    
    if previewScreenGui then
        previewScreenGui:Destroy()
        previewScreenGui = nil
    end
    
    local character = LP.Character
    local humanoid = character:FindFirstChild("Humanoid")
    
    if not humanoid then
        return false
    end
    
    -- Clona o personagem
    previewModel = character:Clone()
    previewModel.Parent = workspace
    
    -- Remove scripts
    for _, child in pairs(previewModel:GetDescendants()) do
        if child:IsA("Script") or child:IsA("LocalScript") then
            child:Destroy()
        end
    end
    
    -- Configura o Humanoid
    local previewHumanoid = previewModel:FindFirstChild("Humanoid")
    if previewHumanoid then
        previewHumanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
        previewHumanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        previewHumanoid.AutoRotate = false
        previewHumanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
        previewHumanoid.Jump = false
        previewHumanoid.PlatformStand = true
    end
    
    -- Posiciona o personagem
    previewModel:SetPrimaryPartCFrame(CFrame.new(0, 0, 20))
    
    -- Cria a UI para o preview
    previewScreenGui = Instance.new("ScreenGui")
    previewScreenGui.Parent = LP.PlayerGui
    previewScreenGui.Name = "PreviewGUI"
    previewScreenGui.ResetOnSpawn = false
    previewScreenGui.IgnoreGuiInset = true
    
    -- Cria o ViewportFrame
    viewportFrame = Instance.new("ViewportFrame")
    viewportFrame.Parent = previewScreenGui
    viewportFrame.Size = UDim2.new(0.35, 0, 0.6, 0)
    viewportFrame.Position = UDim2.new(0.325, 0, 0.2, 0)
    viewportFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    viewportFrame.BackgroundTransparency = 0.2
    viewportFrame.BorderSizePixel = 0
    
    -- Adiciona borda arredondada
    local corner = Instance.new("UICorner")
    corner.Parent = viewportFrame
    corner.CornerRadius = UDim.new(0, 12)
    
    -- Adiciona borda
    local border = Instance.new("Frame")
    border.Parent = viewportFrame
    border.Size = UDim2.new(1, 0, 1, 0)
    border.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    border.BackgroundTransparency = 0.92
    border.BorderSizePixel = 0
    border.ZIndex = 0
    
    local borderCorner = Instance.new("UICorner")
    borderCorner.Parent = border
    borderCorner.CornerRadius = UDim.new(0, 12)
    
    -- Adiciona o personagem ao ViewportFrame
    viewportFrame.WorldRoot = previewModel
    
    -- Cria câmera para o preview
    previewCamera = Instance.new("Camera")
    previewCamera.Parent = viewportFrame
    viewportFrame.CurrentCamera = previewCamera
    
    -- Configura a câmera
    previewCamera.CFrame = CFrame.new(
        Vector3.new(0, 1.8, 8),
        Vector3.new(0, 1.5, 0)
    )
    previewCamera.FieldOfView = 45
    
    -- Fundo gradiente
    local background = Instance.new("Frame")
    background.Parent = viewportFrame
    background.Size = UDim2.new(1, 0, 1, 0)
    background.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    background.BackgroundTransparency = 0.5
    background.ZIndex = 0
    
    local gradient = Instance.new("UIGradient")
    gradient.Parent = background
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 35, 60)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(15, 15, 25))
    })
    
    -- Iluminação
    local light = Instance.new("PointLight")
    light.Parent = viewportFrame
    light.Position = Vector3.new(0, 5, 5)
    light.Color = Color3.fromRGB(255, 255, 255)
    light.Brightness = 2
    light.Range = 20
    
    local light2 = Instance.new("PointLight")
    light2.Parent = viewportFrame
    light2.Position = Vector3.new(-3, 2, 3)
    light2.Color = Color3.fromRGB(200, 180, 255)
    light2.Brightness = 1
    light2.Range = 15
    
    local shadow = Instance.new("DirectionalLight")
    shadow.Parent = viewportFrame
    shadow.Brightness = 0.3
    shadow.Color = Color3.fromRGB(150, 150, 200)
    
    isPreviewActive = true
    previewCreated = true
    rotationAngle = 0
    
    -- Rotação automática
    task.spawn(function()
        while isPreviewActive and viewportFrame and previewModel do
            rotationAngle = rotationAngle + 0.4
            if previewModel.PrimaryPart then
                previewModel:SetPrimaryPartCFrame(
                    CFrame.new(0, 0, 20) * CFrame.Angles(0, math.rad(rotationAngle), 0)
                )
            end
            task.wait(0.05)
        end
    end)
    
    return true
end

local function updatePreview()
    if not isPreviewActive or not previewModel then
        return
    end
    
    local character = LP.Character
    if not character then
        return
    end
    
    -- Remove modelo antigo
    if previewModel then
        previewModel:Destroy()
        previewModel = nil
    end
    
    -- Cria novo modelo atualizado
    previewModel = character:Clone()
    previewModel.Parent = workspace
    
    -- Remove scripts
    for _, child in pairs(previewModel:GetDescendants()) do
        if child:IsA("Script") or child:IsA("LocalScript") then
            child:Destroy()
        end
    end
    
    -- Configura Humanoid
    local previewHumanoid = previewModel:FindFirstChild("Humanoid")
    if previewHumanoid then
        previewHumanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
        previewHumanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        previewHumanoid.AutoRotate = false
        previewHumanoid.PlatformStand = true
    end
    
    -- Posiciona com rotação atual
    previewModel:SetPrimaryPartCFrame(
        CFrame.new(0, 0, 20) * CFrame.Angles(0, math.rad(rotationAngle), 0)
    )
    
    -- Atualiza ViewportFrame
    if viewportFrame then
        viewportFrame.WorldRoot = previewModel
    end
    
    -- Mantém câmera
    if previewCamera then
        previewCamera.CFrame = CFrame.new(
            Vector3.new(0, 1.8, 8),
            Vector3.new(0, 1.5, 0)
        )
    end
end

local function destroyPreview()
    isPreviewActive = false
    previewCreated = false
    
    if previewModel then
        previewModel:Destroy()
        previewModel = nil
    end
    
    if previewScreenGui then
        previewScreenGui:Destroy()
        previewScreenGui = nil
    end
    
    viewportFrame = nil
    previewCamera = nil
end

local function recreatePreview()
    if previewCreated then
        destroyPreview()
        task.wait(0.5)
        createPreview()
    end
end

-- Detecta respawn do personagem
LP.CharacterAdded:Connect(function(character)
    task.wait(1)
    if previewCreated then
        recreatePreview()
    end
end)

local MASKS = {
    "maski", "maska", "maskb", "maskc", "maskd",
    "maske", "maskf", "maskg", "maskh", "maskj"
}

-- Variáveis para armazenar os valores atuais
local currentHairColor = Color3.fromRGB(255, 255, 255)
local currentSkinColor = Color3.fromRGB(255, 219, 172)

-- ================= HOME TAB =================
Window:CreateHomeTab({
    SupportedExecutors = {
        "Synapse X", "Krnl", "ProtoSmasher", "Fluxus",
        "Script-Ware", "EasyExploits", "Electron", "JJSploit",
        "Calamari", "SirHurt", "Sentinel", "WEAREDEVS",
        "Comet", "Cellery", "Wave", "CODex", "Delta"
    },
    DiscordInvite = "oracore",
    Icon = 1
})

-- ================= TAB DE PREVIEW 3D =================
local PreviewTab = Window:CreateTab({
    Name = "🎮 Preview 3D",
    Icon = "3d_rotation",
    ImageSource = "Material",
    ShowTitle = true
})

PreviewTab:CreateSection("👤 Visualização 3D")

PreviewTab:CreateParagraph({
    Title = "📖 Sobre o Preview",
    Text = "Seu personagem em 3D está SEMPRE ATIVO!\n\n" ..
           "• Visualização em tempo real\n" ..
           "• Gira automaticamente\n" ..
           "• Atualiza com todas as alterações\n" ..
           "• Recria automaticamente após respawn"
})

PreviewTab:CreateButton({
    Name = "🔄 Recriar Preview",
    Callback = function()
        recreatePreview()
    end
})

-- ================= TAB DE CORES =================
local ColorsTab = Window:CreateTab({
    Name = "🎨 Cores",
    Icon = "palette",
    ImageSource = "Material",
    ShowTitle = true
})

-- Seção Cabelo
ColorsTab:CreateSection("💇‍♂️ Cabelo")

local HairColorInput = ColorsTab:CreateInput({
    Name = "RGB Manual",
    PlaceholderText = "255,255,255",
    CurrentValue = "255,255,255",
    Numeric = false,
    Callback = function(Text)
        local r, g, b = Text:match("(%d+),(%d+),(%d+)")
        if r and g and b then
            local color = Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b))
            currentHairColor = color
            pcall(function()
                HairColorPicker:SetColor(color)
            end)
        end
    end
}, "Input")

local HairColorPicker = ColorsTab:CreateColorPicker({
    Name = "Cor do Cabelo",
    Color = Color3.fromRGB(255, 255, 255),
    Flag = "HairColor",
    Callback = function(Color)
        if Color then
            currentHairColor = Color
            local r = math.floor(Color.R * 255)
            local g = math.floor(Color.G * 255)
            local b = math.floor(Color.B * 255)
            pcall(function()
                HairColorInput:SetText(string.format("%d,%d,%d", r, g, b))
            end)
        end
    end
}, "ColorPicker")

ColorsTab:CreateButton({
    Name = "Aplicar Cor do Cabelo",
    Callback = function()
        local color = currentHairColor
        local r = math.floor(color.R * 255)
        local g = math.floor(color.G * 255)
        local b = math.floor(color.B * 255)
        sendColor("haircolor", r, g, b)
    end
})

-- Divider e Seção Pele
ColorsTab:CreateDivider()
ColorsTab:CreateSection("👤 Pele")

local SkinColorInput = ColorsTab:CreateInput({
    Name = "RGB Manual",
    PlaceholderText = "255,219,172",
    CurrentValue = "255,219,172",
    Numeric = false,
    Callback = function(Text)
        local r, g, b = Text:match("(%d+),(%d+),(%d+)")
        if r and g and b then
            local color = Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b))
            currentSkinColor = color
            pcall(function()
                SkinColorPicker:SetColor(color)
            end)
        end
    end
}, "Input")

local SkinColorPicker = ColorsTab:CreateColorPicker({
    Name = "Cor da Pele",
    Color = Color3.fromRGB(255, 219, 172),
    Flag = "SkinColor",
    Callback = function(Color)
        if Color then
            currentSkinColor = Color
            local r = math.floor(Color.R * 255)
            local g = math.floor(Color.G * 255)
            local b = math.floor(Color.B * 255)
            pcall(function()
                SkinColorInput:SetText(string.format("%d,%d,%d", r, g, b))
            end)
        end
    end
}, "ColorPicker")

ColorsTab:CreateButton({
    Name = "Aplicar Cor da Pele",
    Callback = function()
        local color = currentSkinColor
        local r = math.floor(color.R * 255)
        local g = math.floor(color.G * 255)
        local b = math.floor(color.B * 255)
        sendColor("skin", r, g, b)
    end
})

-- ================= TAB DE VESTUÁRIO =================
local ClothingTab = Window:CreateTab({
    Name = "👕 Vestuário",
    Icon = "checkroom",
    ImageSource = "Material",
    ShowTitle = true
})

-- Seção Roupas
ClothingTab:CreateSection("👔 Roupas")

local ShirtInput = ClothingTab:CreateInput({
    Name = "ID da Camisa",
    PlaceholderText = "123456789",
    CurrentValue = "",
    Numeric = true,
    Callback = function(Text) end
}, "Input")

local PantsInput = ClothingTab:CreateInput({
    Name = "ID da Calça",
    PlaceholderText = "987654321",
    CurrentValue = "",
    Numeric = true,
    Callback = function(Text) end
}, "Input")

ClothingTab:CreateButton({
    Name = "Aplicar Roupa",
    Callback = function()
        local shirt = tonumber(ShirtInput:GetValue()) or 0
        local pants = tonumber(PantsInput:GetValue()) or 0
        if shirt > 0 or pants > 0 then
            applyCustomOutfit(shirt, pants)
        end
    end
})

ClothingTab:CreateButton({
    Name = "Resetar Roupa",
    Callback = resetOutfit
})

-- Seção Máscaras
ClothingTab:CreateDivider()
ClothingTab:CreateSection("🎭 Máscaras")

local MaskDropdown = ClothingTab:CreateDropdown({
    Name = "Selecionar Máscara",
    Options = MASKS,
    CurrentOption = {MASKS[1]},
    MultipleOptions = false,
    Callback = function(Options)
        if Options and #Options > 0 then
            applyMask(Options[1])
        end
    end
}, "Dropdown")

ClothingTab:CreateButton({
    Name = "Aplicar Mask I",
    Callback = function() 
        applyMask("maski")
        pcall(function()
            MaskDropdown:SetCurrentOption({"maski"})
        end)
    end
})

ClothingTab:CreateButton({
    Name = "Aplicar Mask D",
    Callback = function()
        applyMask("maskd")
        pcall(function()
            MaskDropdown:SetCurrentOption({"maskd"})
        end)
    end
})

ClothingTab:CreateButton({
    Name = "Remover Máscara",
    Callback = function()
        applyMask("nomask")
        pcall(function()
            MaskDropdown:SetCurrentOption({})
        end)
    end
})

-- ================= TAB DE CONFIGURAÇÕES =================
local SettingsTab = Window:CreateTab({
    Name = "⚙️ Configurações",
    Icon = "settings",
    ImageSource = "Material",
    ShowTitle = true
})

SettingsTab:BuildThemeSection()
SettingsTab:BuildConfigSection()

-- ================= INICIAR PREVIEW AUTOMATICAMENTE =================
task.spawn(function()
    task.wait(2)
    createPreview()
end)

-- ================= CARREGAR CONFIGURAÇÕES =================
task.spawn(function()
    task.wait(1)
    pcall(function()
        if Window.LoadConfig then
            Window:LoadConfig()
        end
    end)
end)

-- ================= EXPORTAR FUNÇÕES =================
_G.OraCore = {
    sendColor = sendColor,
    applyMask = applyMask,
    applyCustomOutfit = applyCustomOutfit,
    resetOutfit = resetOutfit,
    createPreview = createPreview,
    destroyPreview = destroyPreview,
    updatePreview = updatePreview,
    recreatePreview = recreatePreview,
    MASKS = MASKS,
    Window = Window,
    Luna = Luna
}

print("Ora Core carregado com sucesso!")
