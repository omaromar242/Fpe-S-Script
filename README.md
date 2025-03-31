-- Load LinoriaLib from GitHub
local repo = 'https://raw.githubusercontent.com/violin-suzutsuki/LinoriaLib/main/'
local Library = loadstring(game:HttpGet(repo .. 'Library.lua'))()

-- Create the main window
local Window = Library:CreateWindow({
    Title = 'Example Menu',
    Center = true,
    AutoShow = true,
    TabPadding = 8,
    MenuFadeTime = 0.2
})

-- Create Tabs: Main and UI Settings
local Tabs = {
    Main = Window:AddTab('Main'),
    ['UI Settings'] = Window:AddTab('UI Settings'),
}

---------------------------------------------
-- MAIN TAB - Fpe:S Group for our toggles
---------------------------------------------
local FpeSGroup = Tabs.Main:AddLeftGroupbox('Fpe:S')

-- Services & local player reference
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

---------------------------
-- ESP Students/Teachers
---------------------------
local ESPEnabled = false

local function applyESPOutline(player)
    if player == LocalPlayer then return end
    local character = player.Character
    if character and not character:FindFirstChild("FpeHighlight") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "FpeHighlight"
        highlight.Adornee = character
        highlight.FillTransparency = 1       -- Outline only
        highlight.OutlineTransparency = 0    -- Fully visible outline
        highlight.OutlineColor = player.TeamColor.Color  -- Use team color
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- See through walls
        highlight.Parent = character
    end
end

local function removeESPOutline(player)
    local character = player.Character
    if character then
        local h = character:FindFirstChild("FpeHighlight")
        if h then
            h:Destroy()
        end
    end
end

local function updateAllESP(apply)
    for _, player in pairs(Players:GetPlayers()) do
        if apply then
            applyESPOutline(player)
        else
            removeESPOutline(player)
        end
    end
end

local function connectCharacter(player)
    player.CharacterAdded:Connect(function(character)
        if ESPEnabled then
            wait(0.5)
            applyESPOutline(player)
        end
    end)
end

for _, player in pairs(Players:GetPlayers()) do
    connectCharacter(player)
end
Players.PlayerAdded:Connect(connectCharacter)

FpeSGroup:AddToggle('EspStudentsTeachers', {
    Text = 'ESP Students/Teachers',
    Default = false,
    Tooltip = 'Toggle outline-only ESP with team color (see through walls)',
    Callback = function(Value)
        ESPEnabled = Value
        updateAllESP(ESPEnabled)
        print('[ESP] Students/Teachers is now:', Value)
    end
})

---------------------------
-- Hitbox All Toggle
---------------------------
local HitboxEnabled = false
local hitboxLoopActive = false

local function updateHitboxes()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                local hrp = character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.Size = Vector3.new(10, 10, 10)
                end
            end
        end
    end
end

FpeSGroup:AddToggle('HitboxAll', {
    Text = 'Hitbox All',
    Default = false,
    Tooltip = 'Every 2 seconds, set all other players HumanoidRootPart size to (10,10,10)',
    Callback = function(Value)
        HitboxEnabled = Value
        if HitboxEnabled and not hitboxLoopActive then
            hitboxLoopActive = true
            spawn(function()
                while hitboxLoopActive do
                    updateHitboxes()
                    wait(2)
                end
            end)
        elseif not HitboxEnabled then
            hitboxLoopActive = false
        end
        print('[Hitbox All] is now:', Value)
    end
})

---------------------------------------------
-- UI SETTINGS TAB - Basic Menu Options
---------------------------------------------
local MenuGroup = Tabs['UI Settings']:AddLeftGroupbox('Menu')
MenuGroup:AddButton('Unload', function() Library:Unload() end)
MenuGroup:AddLabel('Menu bind'):AddKeyPicker('MenuKeybind', {
    Default = 'End',
    NoUI = true,
    Text = 'Menu keybind'
})
Library.ToggleKeybind = Options.MenuKeybind

---------------------------------------------
-- Cleanup on unload: remove active ESP highlights
---------------------------------------------
Library:OnUnload(function()
    updateAllESP(false)
    hitboxLoopActive = false
    print('Script unloaded, ESP and hitbox modifications removed.')
end)
