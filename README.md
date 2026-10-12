
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local old = playerGui:FindFirstChild("ZombieControlUltimate")
if old then old:Destroy() end

--========================================================
-- ZOMBIE DATABASE
--========================================================

local zombieNames = {
    "Zombie", "Fast Zombie", "Explosive Zombie", "Slime",
    "Sand Monster", "Ice Zombie", "Lava Zombie", "Void Slime",
    "Vampire", "Ghost", "Angel", "Demon", "Archangel",
    "Beast", "Crystal", "Santa", "Predator Zombie",
    "Tank Zombie", "Tank Fast Zombie", "Tank Explosive Zombie",
    "Tank Slime", "Tank Sand Monster", "Tank Ice Zombie",
    "Tank Lava Zombie", "Tank Void Slime", "Tank Vampire",
    "Tank Ghost", "Tank Angel", "Tank Demon", "Tank Archangel",
    "Tank Beast", "Tank Crystal", "Tank Santa", "Tank Alien",
    "Tank Monsters", "Enraged Zombie", "Enraged Fast Zombie",
    "Enraged Explosive Zombie", "Enraged Slime",
    "Enraged Sand Monster", "Enraged Ice Zombie",
    "Enraged Lava Zombie", "Enraged Void Slime",
    "Enraged Vampire", "Enraged Ghost", "Enraged Angel",
    "Enraged Demon", "Enraged Archangel", "Enraged Beast",
    "Enraged Crystal", "Enraged Santa",
    "Alien", "Fast Alien", "Explosive Alien",
    "Tank Fast Alien", "Tank Explosive Alien",
    "Enraged Alien", "Enraged Fast Alien", "Enraged Explosive Alien",
    "Pumpkin Zombie", "Fast Pumpkin Zombie",
    "Explosive Pumpkin Zombie", "Tank Pumpkin Zombie",
    "Enraged Pumpkin Zombie",
    "Elf", "Fast Elf", "Explosive Elf", "Tank Elf",
    "Enraged Elf", "Elf Raiders", "Wingmen", "Elves",
    "Mega Tank", "King Slime", "Dark Ghost", "Demon Overlord",
    "Dragon Beast", "Alien Leader", "Aliens", "Zombie Horde",
    "Enraged Monsters"
}

local zombieSet = {}
for _, name in ipairs(zombieNames) do
    zombieSet[name] = true
end

--========================================================
-- SETTINGS
--========================================================

local settings = {
    multiplier = 2,
    esp = true,
    names = true,
    health = true,
    distance = false,
    maxDistance = 500,
    outlineTransparency = 0.1,
    fillTransparency = 0.9,
    theme = 1,
    selectedTab = "Zombies",
    minimized = false,
}

local themes = {
    {
        bg = Color3.fromRGB(10, 13, 27),
        panel = Color3.fromRGB(20, 25, 45),
        button = Color3.fromRGB(37, 43, 72),
        accent = Color3.fromRGB(130, 95, 255),
        text = Color3.fromRGB(241, 243, 255),
        muted = Color3.fromRGB(153, 165, 198),
        good = Color3.fromRGB(52, 190, 143),
        bad = Color3.fromRGB(194, 64, 83),
    },
    {
        bg = Color3.fromRGB(16, 22, 25),
        panel = Color3.fromRGB(25, 38, 40),
        button = Color3.fromRGB(39, 59, 59),
        accent = Color3.fromRGB(52, 204, 170),
        text = Color3.fromRGB(237, 250, 244),
        muted = Color3.fromRGB(157, 191, 183),
        good = Color3.fromRGB(52, 190, 143),
        bad = Color3.fromRGB(194, 64, 83),
    },
    {
        bg = Color3.fromRGB(25, 16, 25),
        panel = Color3.fromRGB(43, 25, 43),
        button = Color3.fromRGB(66, 37, 66),
        accent = Color3.fromRGB(255, 93, 174),
        text = Color3.fromRGB(255, 239, 250),
        muted = Color3.fromRGB(202, 155, 190),
        good = Color3.fromRGB(52, 190, 143),
        bad = Color3.fromRGB(194, 64, 83),
    },
}

local function palette()
    return themes[settings.theme]
end

local selected = {}
local tracked = {}
local originalSizes = {}
local espObjects = {}
local listButtons = {}
local presets = {}

local function isZombie(model)
    return model and model:IsA("Model") and zombieSet[model.Name] == true
end

local function getRoot(model)
    return model:FindFirstChild("HumanoidRootPart")
        or model.PrimaryPart
        or model:FindFirstChild("Head")
end

local function getHumanoid(model)
    return model:FindFirstChildOfClass("Humanoid")
end

--========================================================
-- GUI HELPERS
--========================================================

local gui = Instance.new("ScreenGui")
gui.Name = "ZombieControlUltimate"
gui.ResetOnSpawn = false
gui.DisplayOrder = 100
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

local function corner(obj, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 9)
    c.Parent = obj
    return c
end

local function outline(obj, color, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color or palette().accent
    s.Transparency = transparency or 0.3
    s.Thickness = 1
    s.Parent = obj
    return s
end

local function addPadding(obj, amount)
    local p = Instance.new("UIPadding")
    p.PaddingTop = UDim.new(0, amount)
    p.PaddingBottom = UDim.new(0, amount)
    p.PaddingLeft = UDim.new(0, amount)
    p.PaddingRight = UDim.new(0, amount)
    p.Parent = obj
end

local function newLabel(parent, text, size, color)
    local l = Instance.new("TextLabel")
    l.BackgroundTransparency = 1
    l.Text = text or ""
    l.TextColor3 = color or palette().text
    l.Font = Enum.Font.Gotham
    l.TextSize = size or 13
    l.TextWrapped = true
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = parent
    return l
end

local function newButton(parent, text, callback)
    local b = Instance.new("TextButton")
    b.Text = text
    b.TextColor3 = palette().text
    b.BackgroundColor3 = palette().button
    b.Font = Enum.Font.GothamSemibold
    b.TextSize = 13
    b.TextWrapped = true
    b.AutoButtonColor = true
    b.Parent = parent
    corner(b, 8)
    outline(b, palette().accent, 0.65)
    if callback then
        b.Activated:Connect(callback)
    end
    return b
end

local function newPanel(parent, name, height)
    local f = Instance.new("Frame")
    f.Name = name
    f.Size = UDim2.new(1, 0, 0, height)
    f.BackgroundColor3 = palette().panel
    f.Parent = parent
    corner(f, 10)
    outline(f, palette().accent, 0.7)
    addPadding(f, 9)
    return f
end

local function setStatus(text)
    if gui.Parent and statusLabel then
        statusLabel.Text = "● " .. text
    end
end

--========================================================
-- MAIN WINDOW
--========================================================

local main = Instance.new("Frame")
main.Name = "MainWindow"
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.Position = UDim2.fromScale(0.5, 0.5)
main.Size = UDim2.new(0.92, 0, 0.84, 0)
main.BackgroundColor3 = palette().bg
main.ClipsDescendants = true
main.Parent = gui
corner(main, 14)
outline(main, palette().accent, 0.1)

local sizeLimit = Instance.new("UISizeConstraint")
sizeLimit.MinSize = Vector2.new(285, 310)
sizeLimit.MaxSize = Vector2.new(900, 720)
sizeLimit.Parent = main

-- Header is also the drag handle.
local header = Instance.new("Frame")
header.Name = "Header"
header.Size = UDim2.new(1, 0, 0, 48)
header.BackgroundColor3 = palette().panel
header.Parent = main
corner(header, 12)

local title = newLabel(header, "◈  ZOMBIE CONTROL", 17, palette().accent)
title.Font = Enum.Font.GothamBold
title.Position = UDim2.fromOffset(12, 0)
title.Size = UDim2.new(1, -125, 1, 0)

local minimizeButton = newButton(header, "—")
minimizeButton.Size = UDim2.fromOffset(36, 32)
minimizeButton.Position = UDim2.new(1, -82, 0.5, -16)

local closeButton = newButton(header, "×")
closeButton.Size = UDim2.fromOffset(36, 32)
closeButton.Position = UDim2.new(1, -41, 0.5, -16)

local statusLabel = newLabel(main, "● Starting...", 11, palette().good)
statusLabel.Position = UDim2.new(0, 10, 0, 51)
statusLabel.Size = UDim2.new(1, -20, 0, 20)

-- Navigation
local nav = Instance.new("ScrollingFrame")
nav.Name = "Navigation"
nav.Position = UDim2.fromOffset(8, 75)
nav.Size = UDim2.new(1, -16, 0, 37)
nav.BackgroundTransparency = 1
nav.BorderSizePixel = 0
nav.ScrollBarThickness = 2
nav.ScrollingDirection = Enum.ScrollingDirection.X
nav.AutomaticCanvasSize = Enum.AutomaticSize.X
nav.CanvasSize = UDim2.new()
nav.Parent = main

local navLayout = Instance.new("UIListLayout")
navLayout.FillDirection = Enum.FillDirection.Horizontal
navLayout.Padding = UDim.new(0, 5)
navLayout.Parent = nav

local body = Instance.new("Frame")
body.Name = "Body"
body.Position = UDim2.fromOffset(8, 119)
body.Size = UDim2.new(1, -16, 1, -127)
body.BackgroundTransparency = 1
body.Parent = main

local pages = {}
local navButtons = {}

local function createPage(name)
    local page = Instance.new("ScrollingFrame")
    page.Name = name .. "Page"
    page.Size = UDim2.fromScale(1, 1)
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.ScrollBarThickness = 4
    page.ScrollBarImageColor3 = palette().accent
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.CanvasSize = UDim2.new()
    page.Visible = false
    page.Parent = body
    addPadding(page, 2)

    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = page

    pages[name] = page

    local navButton = newButton(nav, name)
    navButton.Size = UDim2.fromOffset(105, 33)
    navButtons[name] = navButton

    navButton.Activated:Connect(function()
        settings.selectedTab = name
        for pageName, p in pairs(pages) do
            p.Visible = pageName == name
        end
        for pageName, b in pairs(navButtons) do
            b.BackgroundColor3 = pageName == name
                and palette().accent or palette().button
        end
    end)

    return page
end

local zombiePage = createPage("Zombies")
local espPage = createPage("ESP")
local settingsPage = createPage("Settings")
local aboutPage = createPage("About")

local function showPage(name)
    settings.selectedTab = name
    for pageName, page in pairs(pages) do
        page.Visible = pageName == name
    end
    for pageName, b in pairs(navButtons) do
        b.BackgroundColor3 = pageName == name
            and palette().accent or palette().button
    end
end

--========================================================
-- ZOMBIE TAB
--========================================================

local search = Instance.new("TextBox")
search.Name = "Search"
search.Size = UDim2.new(1, 0, 0, 38)
search.BackgroundColor3 = palette().button
search.TextColor3 = palette().text
search.PlaceholderColor3 = palette().muted
search.PlaceholderText = "Search zombie names..."
search.Text = ""
search.ClearTextOnFocus = false
search.Font = Enum.Font.Gotham
search.TextSize = 14
search.Parent = zombiePage
corner(search, 8)
outline(search, palette().accent, 0.65)
addPadding(search, 8)

local listPanel = newPanel(zombiePage, "ZombieList", 250)
local listTitle = newLabel(listPanel, "ZOMBIE DATABASE", 13, palette().accent)
listTitle.Size = UDim2.new(1, 0, 0, 22)

local countLabel = newLabel(listPanel, "0 selected", 11, palette().muted)
countLabel.Position = UDim2.fromOffset(0, 22)
countLabel.Size = UDim2.new(1, 0, 0, 18)

local zombieScroll = Instance.new("ScrollingFrame")
zombieScroll.Position = UDim2.fromOffset(0, 43)
zombieScroll.Size = UDim2.new(1, 0, 1, -43)
zombieScroll.BackgroundTransparency = 1
zombieScroll.BorderSizePixel = 0
zombieScroll.ScrollBarThickness = 4
zombieScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
zombieScroll.CanvasSize = UDim2.new()
zombieScroll.Parent = listPanel

local zombieLayout = Instance.new("UIListLayout")
zombieLayout.Padding = UDim.new(0, 4)
zombieLayout.Parent = zombieScroll

local function updateCount()
    local n = 0
    for _ in pairs(selected) do n += 1 end
    countLabel.Text = n .. " selected  |  " .. #zombieNames .. " types"
end

local function updateFilter()
    local q = string.lower(search.Text)
    for name, b in pairs(listButtons) do
        b.Visible = string.find(string.lower(name), q, 1, true) ~= nil
    end
end

local function updateZombieButton(name)
    local b = listButtons[name]
    if not b then return end
    b.Text = (selected[name] and "✓   " or "+   ") .. name
    b.BackgroundColor3 = selected[name]
        and palette().accent or palette().button
end

for index, name in ipairs(zombieNames) do
    local b = newButton(zombieScroll, "+   " .. name)
    b.Name = "Zombie_" .. index
    b.Size = UDim2.new(1, -4, 0, 32)
    b.LayoutOrder = index
    b.TextXAlignment = Enum.TextXAlignment.Left
    listButtons[name] = b

    b.Activated:Connect(function()
        selected[name] = selected[name] and nil or true
        updateZombieButton(name)
        updateCount()
    end)
end

search:GetPropertyChangedSignal("Text"):Connect(updateFilter)

local quickPanel = newPanel(zombiePage, "QuickActions", 130)
local quickTitle = newLabel(quickPanel, "QUICK ACTIONS", 13, palette().accent)
quickTitle.Size = UDim2.new(1, 0, 0, 22)

local function actionButton(parent, text, x, y, width, callback)
    local b = newButton(parent, text, callback)
    b.Position = UDim2.new(x, 0, 0, y)
    b.Size = UDim2.new(width, -4, 0, 34)
    return b
end

actionButton(quickPanel, "Select All", 0, 29, 0.33, function()
    for _, name in ipairs(zombieNames) do
        selected[name] = true
        updateZombieButton(name)
    end
    updateCount()
    setStatus("All zombie types selected")
end)

actionButton(quickPanel, "Clear", 0.333, 29, 0.33, function()
    table.clear(selected)
    for _, name in ipairs(zombieNames) do updateZombieButton(name) end
    updateCount()
    setStatus("Selection cleared")
end)

actionButton(quickPanel, "Refresh", 0.666, 29, 0.334, function()
    local count = 0
    for model in pairs(tracked) do
        if model and model.Parent then count += 1 end
    end
    setStatus("Cached NPCs: " .. count)
end)

actionButton(quickPanel, "Resize Selected", 0, 70, 0.5, function()
    for model in pairs(tracked) do
        if model and model.Parent and selected[model.Name] then
            resizeModel(model, settings.multiplier)
        end
    end
    setStatus("Selected types resized locally")
end)

actionButton(quickPanel, "Resize All", 0.5, 70, 0.5, function()
    for model in pairs(tracked) do
        if model and model.Parent then
            resizeModel(model, settings.multiplier)
        end
    end
    setStatus("All tracked types resized locally")
end)

local resetPanel = newPanel(zombiePage, "ResetTools", 87)
local resetTitle = newLabel(resetPanel, "RESTORE ORIGINAL SIZES", 13, palette().accent)
resetTitle.Size = UDim2.new(1, 0, 0, 22)

actionButton(resetPanel, "Reset Selected", 0, 30, 0.5, function()
    for model in pairs(tracked) do
        if model and selected[model.Name] then resetModel(model) end
    end
    setStatus("Selected hitboxes restored")
end)

actionButton(resetPanel, "Reset All", 0.5, 30, 0.5, function()
    for model in pairs(tracked) do resetModel(model) end
    setStatus("All saved hitboxes restored")
end)

--========================================================
-- ESP TAB
--========================================================

local espPanel = newPanel(espPage, "ESPControls", 270)
local espTitle = newLabel(espPanel, "ESP VISUAL CONTROLS", 14, palette().accent)
espTitle.Size = UDim2.new(1, 0, 0, 23)

local function toggleRow(parent, text, y, initial, callback)
    local l = newLabel(parent, text, 12)
    l.Position = UDim2.fromOffset(0, y)
    l.Size = UDim2.new(1, -68, 0, 30)

    local state = initial
    local b = newButton(parent, state and "ON" or "OFF", function()
        state = not state
        b.Text = state and "ON" or "OFF"
        b.BackgroundColor3 = state and palette().good or palette().button
        callback(state)
    end)
    b.Position = UDim2.new(1, -60, 0, y + 2)
    b.Size = UDim2.fromOffset(60, 26)
    b.BackgroundColor3 = state and palette().good or palette().button
    return b
end

toggleRow(espPanel, "ESP enabled", 29, settings.esp, function(v)
    settings.esp = v
end)

toggleRow(espPanel, "Show zombie names", 62, settings.names, function(v)
    settings.names = v
end)

toggleRow(espPanel, "Show health text", 95, settings.health, function(v)
    settings.health = v
end)

toggleRow(espPanel, "Show distance", 128, settings.distance, function(v)
    settings.distance = v
end)

local colorButton = newButton(espPanel, "Cycle outline color")
colorButton.Position = UDim2.fromOffset(0, 169)
colorButton.Size = UDim2.new(1, 0, 0, 33)

local espColors = {
    Color3.fromRGB(130, 95, 255),
    Color3.fromRGB(60, 240, 140),
    Color3.fromRGB(255, 75, 95),
    Color3.fromRGB(50, 195, 255),
    Color3.fromRGB(255, 177, 60),
}
local colorIndex = 1
settings.outlineColor = espColors[colorIndex]

colorButton.Activated:Connect(function()
    colorIndex = colorIndex % #espColors + 1
    settings.outlineColor = espColors[colorIndex]
    colorButton.Text = "Outline color " .. colorIndex
    colorButton.BackgroundColor3 = settings.outlineColor
end)

local distanceButton = newButton(espPanel, "Max distance: 500 studs")
distanceButton.Position = UDim2.fromOffset(0, 208)
distanceButton.Size = UDim2.new(1, 0, 0, 33)

distanceButton.Activated:Connect(function()
    if settings.maxDistance == 500 then
        settings.maxDistance = 1000
    elseif settings.maxDistance == 1000 then
        settings.maxDistance = 250
    else
        settings.maxDistance = 500
    end
    distanceButton.Text = "Max distance: " .. settings.maxDistance .. " studs"
    for _, data in pairs(espObjects) do
        if data.billboard then
            data.billboard.MaxDistance = settings.maxDistance
        end
    end
end)

--========================================================
-- SETTINGS TAB
--========================================================

local sizePanel = newPanel(settingsPage, "HitboxSettings", 155)
local sizeTitle = newLabel(sizePanel, "HITBOX MULTIPLIER", 14, palette().accent)
sizeTitle.Size = UDim2.new(1, 0, 0, 23)

local multiplierLabel = newLabel(sizePanel, "Current: 2.0x", 13)
multiplierLabel.Position = UDim2.fromOffset(0, 28)
multiplierLabel.Size = UDim2.new(1, 0, 0, 24)

local slider = Instance.new("Frame")
slider.Position = UDim2.fromOffset(0, 58)
slider.Size = UDim2.new(1, 0, 0, 24)
slider.BackgroundColor3 = palette().button
slider.Parent = sizePanel
corner(slider, 8)

local fill = Instance.new("Frame")
fill.Size = UDim2.new((settings.multiplier - 0.5) / 9.5, 0, 1, 0)
fill.BackgroundColor3 = palette().accent
fill.Parent = slider
corner(fill, 8)

local sliderHitbox = Instance.new("TextButton")
sliderHitbox.Size = UDim2.fromScale(1, 1)
sliderHitbox.BackgroundTransparency = 1
sliderHitbox.Text = ""
sliderHitbox.Parent = slider

local draggingSlider = false

local function setMultiplier(x)
    local width = math.max(slider.AbsoluteSize.X, 1)
    local alpha = math.clamp(
        (x - slider.AbsolutePosition.X) / width, 0, 1
    )
    settings.multiplier = math.floor((0.5 + alpha * 9.5) * 10 + 0.5) / 10
    fill.Size = UDim2.new(alpha, 0, 1, 0)
    multiplierLabel.Text = "Current: " .. string.format("%.1f", settings.multiplier) .. "x"
end

sliderHitbox.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = true
        setMultiplier(input.Position.X)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingSlider and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        setMultiplier(input.Position.X)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = false
    end
end)

local themePanel = newPanel(settingsPage, "Appearance", 110)
local themeTitle = newLabel(themePanel, "APPEARANCE", 14, palette().accent)
themeTitle.Size = UDim2.new(1, 0, 0, 23)

local themeButton = newButton(themePanel, "Change theme")
themeButton.Position = UDim2.fromOffset(0, 31)
themeButton.Size = UDim2.new(1, 0, 0, 35)

local function applyTheme()
    local p = palette()
    main.BackgroundColor3 = p.bg
    header.BackgroundColor3 = p.panel
    title.TextColor3 = p.accent
    statusLabel.TextColor3 = p.good

    for _, page in pairs(pages) do
        page.ScrollBarImageColor3 = p.accent
    end

    for _, b in pairs(navButtons) do
        b.BackgroundColor3 = p.button
    end
    navButtons[settings.selectedTab].BackgroundColor3 = p.accent

    for _, obj in ipairs(gui:GetDescendants()) do
        if obj:IsA("Frame") and obj ~= main and obj ~= body
            and obj ~= header and obj ~= slider and obj ~= fill then
            if obj.Name ~= "Info" then
                obj.BackgroundColor3 = p.panel
            end
        elseif obj:IsA("TextLabel") then
            obj.TextColor3 = p.text
        elseif obj:IsA("TextButton") then
            obj.TextColor3 = p.text
            if obj ~= themeButton then
                obj.BackgroundColor3 = p.button
            end
        elseif obj:IsA("TextBox") then
            obj.BackgroundColor3 = p.button
            obj.TextColor3 = p.text
        end
    end

    fill.BackgroundColor3 = p.accent
    slider.BackgroundColor3 = p.button
    for name in pairs(listButtons) do updateZombieButton(name) end
    themeButton.BackgroundColor3 = p.accent
end

themeButton.Activated:Connect(function()
    settings.theme = settings.theme % #themes + 1
    applyTheme()
    setStatus("Theme changed")
end)

local presetPanel = newPanel(settingsPage, "SessionPresets", 125)
local presetTitle = newLabel(presetPanel, "SESSION PRESETS", 14, palette().accent)
presetTitle.Size = UDim2.new(1, 0, 0, 23)

local savePresetButton = newButton(presetPanel, "Save current settings")
savePresetButton.Position = UDim2.fromOffset(0, 31)
savePresetButton.Size = UDim2.new(1, 0, 0, 33)

local loadPresetButton = newButton(presetPanel, "Load saved settings")
loadPresetButton.Position = UDim2.fromOffset(0, 70)
loadPresetButton.Size = UDim2.new(1, 0, 0, 33)

savePresetButton.Activated:Connect(function()
    presets.default = {
        multiplier = settings.multiplier,
        esp = settings.esp,
        names = settings.names,
        health = settings.health,
        distance = settings.distance,
        maxDistance = settings.maxDistance,
        outlineColor = settings.outlineColor,
        outlineTransparency = settings.outlineTransparency,
    }
    setStatus("Preset saved for this session")
end)

loadPresetButton.Activated:Connect(function()
    local p = presets.default
    if not p then
        setStatus("Save a preset first")
        return
    end
    for key, value in pairs(p) do settings[key] = value end
    multiplierLabel.Text = "Current: " .. string.format("%.1f", settings.multiplier) .. "x"
    fill.Size = UDim2.new((settings.multiplier - 0.5) / 9.5, 0, 1, 0)
    setStatus("Preset loaded")
end)

--========================================================
-- ABOUT TAB
--========================================================

local aboutPanel = newPanel(aboutPage, "AboutPanel", 180)
local aboutTitle = newLabel(aboutPanel, "ZOMBIE CONTROL ULTIMATE", 15, palette().accent)
aboutTitle.Size = UDim2.new(1, 0, 0, 26)

local aboutText = newLabel(
    aboutPanel,
    "Responsive mobile + PC interface\n"
    .. "Searchable zombie database\n"
    .. "Selected and all-zombie controls\n"
    .. "ESP outlines, names, health and distance\n"
    .. "Adjustable hitbox multiplier\n"
    .. "Three themes and session presets\n"
    .. "Cached NPC tracking\n\n"
    .. "Right Shift: toggle window\n"
    .. "Session presets reset when this script restarts.",
    12,
    palette().muted
)
aboutText.Position = UDim2.fromOffset(0, 30)
aboutText.Size = UDim2.new(1, 0, 1, -30)

--========================================================
-- HITBOX HANDLING
--========================================================

function resizeModel(model, multiplier)
    if not model or not model.Parent then return end

    originalSizes[model] = originalSizes[model] or {}

    for _, part in ipairs(model:GetDescendants()) do
        if part:IsA("BasePart") then
            if originalSizes[model][part] == nil then
                originalSizes[model][part] = part.Size
            end

            local original = originalSizes[model][part]
            pcall(function()
                part.Size = original * multiplier
            end)
        end
    end
end

function resetModel(model)
    local saved = originalSizes[model]
    if not saved then return end

    for part, size in pairs(saved) do
        if part and part.Parent then
            pcall(function()
                part.Size = size
            end)
        end
    end
    originalSizes[model] = nil
end

--========================================================
-- ESP MANAGEMENT
--========================================================

local function clearESP(model)
    local data = espObjects[model]
    if not data then return end
    if data.highlight then data.highlight:Destroy() end
    if data.billboard then data.billboard:Destroy() end
    espObjects[model] = nil
end

local function createESP(model)
    if not settings.esp or espObjects[model] or not model.Parent then
        return
    end

    local rootPart = getRoot(model)
    if not rootPart then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "ZCU_Highlight"
    highlight.Adornee = model
    highlight.FillColor = settings.outlineColor or palette().accent
    highlight.OutlineColor = settings.outlineColor or palette().accent
    highlight.FillTransparency = settings.fillTransparency
    highlight.OutlineTransparency = settings.outlineTransparency
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = model

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ZCU_Label"
    billboard.Adornee = rootPart
    billboard.Size = UDim2.fromOffset(180, 48)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.MaxDistance = settings.maxDistance
    billboard.Parent = playerGui

    local text = Instance.new("TextLabel")
    text.Name = "Info"
    text.Size = UDim2.fromScale(1, 1)
    text.BackgroundColor3 = Color3.fromRGB(8, 11, 23)
    text.BackgroundTransparency = 0.2
    text.TextColor3 = settings.outlineColor or palette().accent
    text.TextStrokeTransparency = 0.45
    text.Font = Enum.Font.GothamBold
    text.TextSize = 12
    text.TextWrapped = true
    text.Parent = billboard
    corner(text, 5)

    espObjects[model] = {
        highlight = highlight,
        billboard = billboard,
        text = text,
    }
end

local function track(model)
    if not isZombie(model) or tracked[model] then return end
    tracked[model] = true

    model.AncestryChanged:Connect(function(_, parent)
        if not parent then
            clearESP(model)
            tracked[model] = nil
            originalSizes[model] = nil
        end
    end)

    if settings.esp then
        task.defer(function()
            if model.Parent then createESP(model) end
        end)
    end
end

-- Initial scan once
for _, obj in ipairs(Workspace:GetDescendants()) do
    if isZombie(obj) then track(obj) end
end

-- Track models as they're added
Workspace.DescendantAdded:Connect(function(obj)
    if obj:IsA("Model") and isZombie(obj) then
        track(obj)
    elseif obj:IsA("Model") then
        task.defer(function()
            if obj.Parent and isZombie(obj) then track(obj) end
        end)
    end
end)

--========================================================
-- ESP UPDATE LOOP
--========================================================

local accumulator = 0

RunService.Heartbeat:Connect(function(dt)
    accumulator += dt
    if accumulator < 0.25 then return end
    accumulator = 0

    local character = player.Character
    local playerRoot = character and character:FindFirstChild("HumanoidRootPart")

    for model in pairs(tracked) do
        if not model or not model.Parent then
            clearESP(model)
            tracked[model] = nil
            continue
        end

        if not settings.esp then
            clearESP(model)
            continue
        end

        createESP(model)
        local data = espObjects[model]
        if not data then continue end

        local rootPart = getRoot(model)
        if not rootPart then
            data.billboard.Enabled = false
            data.highlight.Enabled = false
            continue
        end

        local distance = 0
        if playerRoot then
            distance = (playerRoot.Position - rootPart.Position).Magnitude
        end

        local inRange = not playerRoot or distance <= settings.maxDistance
        local humanoid = getHumanoid(model)
        local alive = not humanoid or humanoid.Health > 0

        data.billboard.Enabled = inRange and alive
        data.highlight.Enabled = inRange and alive

        data.highlight.FillColor = settings.outlineColor or palette().accent
        data.highlight.OutlineColor = settings.outlineColor or palette().accent
        data.highlight.FillTransparency = settings.fillTransparency
        data.highlight.OutlineTransparency = settings.outlineTransparency
        data.billboard.MaxDistance = settings.maxDistance

        local lines = {}
        if settings.names then
            table.insert(lines, model.Name)
        end
        if settings.health and humanoid then
            table.insert(lines, string.format(
                "HP: %d / %d",
                math.floor(humanoid.Health),
                math.floor(humanoid.MaxHealth)
            ))
        end
        if settings.distance and playerRoot then
            table.insert(lines, math.floor(distance) .. " studs")
        end

        data.text.Text = table.concat(lines, "\n")
        data.text.Visible = #lines > 0
        data.text.TextColor3 = settings.outlineColor or palette().accent
    end
end)

--========================================================
-- DRAGGABLE WINDOW
--========================================================

local dragging = false
local dragStart
local startPosition

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPosition = main.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if not dragging then return end

    if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

--========================================================
-- MOBILE BUTTON / MINIMIZE / KEYBIND
--========================================================

local floating = newButton(gui, "Z", function()
    main.Visible = true
    settings.minimized = false
    floating.Visible = false
end)
floating.Name = "FloatingToggle"
floating.AnchorPoint = Vector2.new(1, 0.5)
floating.Position = UDim2.new(1, -12, 0.5, 0)
floating.Size = UDim2.fromOffset(48, 48)
floating.TextSize = 20
floating.Visible = false
floating.ZIndex = 50

local function updateMinimized()
    if settings.minimized then
        main.Visible = false
        local camera = Workspace.CurrentCamera
        floating.Visible = UserInputService.TouchEnabled
            or (camera and camera.ViewportSize.X < 700)
    else
        main.Visible = true
        floating.Visible = false
    end
end

minimizeButton.Activated:Connect(function()
    settings.minimized = not settings.minimized
    updateMinimized()
end)

closeButton.Activated:Connect(function()
    settings.minimized = true
    updateMinimized()
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        settings.minimized = not settings.minimized
        updateMinimized()
    end
end)

local function applyResponsiveLayout()
    local camera = Workspace.CurrentCamera
    if not camera then return end

    local width = camera.ViewportSize.X
    if width < 700 then
        main.Size = UDim2.new(0.96, 0, 0.86, 0)
        title.TextSize = 14
        floating.Size = UDim2.fromOffset(48, 48)
    else
        main.Size = UDim2.new(0.78, 0, 0.84, 0)
        title.TextSize = 17
        floating.Size = UDim2.fromOffset(48, 48)
    end

    updateMinimized()
end

if Workspace.CurrentCamera then
    Workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(
        applyResponsiveLayout
    )
end

--========================================================
-- STARTUP
--========================================================

showPage("Zombies")
applyResponsiveLayout()
updateCount()
setStatus("Ready | Mobile + PC | " .. #zombieNames .. " zombie types")

