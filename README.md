--[[
    ORA CORE - Adaptado para Luna Interface Suite (CORRIGIDO)
]]

local Luna = loadstring(game:HttpGet("https://raw.githubusercontent.com/Nebula-Softworks/Luna-Interface-Suite/refs/heads/master/source.lua", true))()

local Players = game:GetService("Players")
local LP = Players.LocalPlayer

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

-- ================= FUNÇÕES PRINCIPAIS =================

local function sendColor(cmd, r, g, b)
    if not LP:FindFirstChild("startevent") then 
        Luna:Notification({
            Title = "Erro",
            Icon = "error",
            ImageSource = "Material",
            Content = "startevent não encontrado!"
        })
        return false 
    end
    
    LP.startevent:FireServer(
        cmd,
        string.format("%.2f,%.2f,%.2f", r/255, g/255, b/255)
    )
    
    Luna:Notification({
        Title = "Cor Aplicada",
        Icon = "check_circle",
        ImageSource = "Material",
        Content = string.format("RGB: %d,%d,%d", r, g, b)
    })
    
    return true
end

local function applyMask(mask)
    if LP:FindFirstChild("startevent") then
        LP.startevent:FireServer("mask", mask)
        Luna:Notification({
            Title = "Máscara Aplicada",
            Icon = "face",
            ImageSource = "Material",
            Content = "Máscara: " .. mask
        })
        return true
    end
    return false
end

local function applyCustomOutfit(shirtId, pantsId)
    if LP:FindFirstChild("startevent") then
        LP.startevent:FireServer("customoutfit", shirtId, pantsId)
        Luna:Notification({
            Title = "Roupa Aplicada",
            Icon = "check_circle",
            ImageSource = "Material",
            Content = "Camisa: " .. shirtId .. " | Calça: " .. pantsId
        })
        return true
    end
    return false
end

local function resetOutfit()
    if LP:FindFirstChild("startevent") then
        LP.startevent:FireServer("resetoutfit")
        Luna:Notification({
            Title = "Roupa Resetada",
            Icon = "refresh",
            ImageSource = "Material",
            Content = "Roupa padrão restaurada!"
        })
        return true
    end
    return false
end

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

-- ================= TAB DE CORES =================
local ColorsTab = Window:CreateTab({
    Name = "🎨 Cores",
    Icon = "palette",
    ImageSource = "Material",
    ShowTitle = true
})

-- Seção Cabelo
ColorsTab:CreateSection("💇‍♂️ Cabelo")

-- Color Picker do Cabelo
local HairColorPicker = ColorsTab:CreateColorPicker({
    Name = "Cor do Cabelo",
    Color = Color3.fromRGB(255, 255, 255),
    Flag = "HairColor",
    Callback = function(Color)
        currentHairColor = Color
        -- Atualiza o input com os valores RGB
        local r = math.floor(Color.R * 255)
        local g = math.floor(Color.G * 255)
        local b = math.floor(Color.B * 255)
        HairColorInput:SetText(string.format("%d,%d,%d", r, g, b))
    end
}, "ColorPicker")

-- Input para digitar RGB manualmente
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
            HairColorPicker:SetColor(color)
        end
    end
}, "Input")

-- Botão Aplicar Cabelo
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

-- Color Picker da Pele
local SkinColorPicker = ColorsTab:CreateColorPicker({
    Name = "Cor da Pele",
    Color = Color3.fromRGB(255, 219, 172),
    Flag = "SkinColor",
    Callback = function(Color)
        currentSkinColor = Color
        local r = math.floor(Color.R * 255)
        local g = math.floor(Color.G * 255)
        local b = math.floor(Color.B * 255)
        SkinColorInput:SetText(string.format("%d,%d,%d", r, g, b))
    end
}, "ColorPicker")

-- Input para RGB da Pele
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
            SkinColorPicker:SetColor(color)
        end
    end
}, "Input")

-- Botão Aplicar Pele
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
        else
            Luna:Notification({
                Title = "Erro",
                Icon = "error",
                ImageSource = "Material",
                Content = "Digite IDs válidos!"
            })
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

-- Dropdown de Máscaras
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

-- Botões rápidos
ClothingTab:CreateButton({
    Name = "Aplicar Mask I",
    Callback = function() 
        applyMask("maski")
        MaskDropdown:SetCurrentOption({"maski"})
    end
})

ClothingTab:CreateButton({
    Name = "Aplicar Mask D",
    Callback = function()
        applyMask("maskd")
        MaskDropdown:SetCurrentOption({"maskd"})
    end
})

ClothingTab:CreateButton({
    Name = "Remover Máscara",
    Callback = function()
        applyMask("nomask")
        MaskDropdown:SetCurrentOption({})
    end
})

-- ================= TAB DE CONFIGURAÇÕES =================
local SettingsTab = Window:CreateTab({
    Name = "⚙️ Configurações",
    Icon = "settings",
    ImageSource = "Material",
    ShowTitle = true
})

-- Tema
SettingsTab:BuildThemeSection()

-- Configurações (Salvar/Carregar)
SettingsTab:BuildConfigSection()

-- ================= NOTIFICAÇÃO INICIAL =================
Luna:Notification({
    Title = "Ora Core",
    Icon = "rocket_launch",
    ImageSource = "Material",
    Content = "Carregado com sucesso! Pressione a tecla para abrir."
})

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
    MASKS = MASKS,
    Window = Window,
    Luna = Luna
}

print("Ora Core carregado com sucesso!")
