
-- Zombie Control GUI
-- LocalScript for testing

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

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
    "Enraged Crystal", "Enraged Santa", "Enraged Alien",
    "Alien", "Fast Alien", "Explosive Alien", "Tank Fast Alien",
    "Tank Explosive Alien", "Enraged Fast Alien",
    "Enraged Explosive Alien", "Pumpkin Zombie",
    "Fast Pumpkin Zombie", "Explosive Pumpkin Zombie",
    "Ice Pumpkin Zombie", "Lava Pumpkin Zombie",
    "Tank Pumpkin Zombie", "Tank Fast Pumpkin Zombie",
    "Tank Explosive Pumpkin Zombie", "Tank Ice Pumpkin Zombie",
    "Tank Lava Pumpkin Zombie", "Enraged Pumpkin Zombie",
    "Enraged Fast Pumpkin Zombie", "Enraged Explosive Pumpkin Zombie",
    "Enraged Ice Pumpkin Zombie", "Enraged Lava Pumpkin Zombie",
    "Elf", "Fast Elf", "Explosive Elf", "Tank Elf",
    "Tank Fast Elf", "Tank Explosive Elf", "Enraged Elf",
    "Enraged Fast Elf", "Enraged Explosive Elf",
    "Mega Tank", "King Slime", "Dark Ghost", "Demon Overlord",
    "Dragon Beast", "Alien Leader", "Elves", "Elf Raiders",
    "Wingmen", "Aliens", "Zombie Horde", "Tank Monsters",
    "Enraged Monsters",
}

local selected = "Zombie"
local multiplier = 2
local espEnabled = false
local originalSizes = {}
local highlights = {}
local billboards = {}

local gui = Instance.new("ScreenGui")
gui.Name = "ZombieControlGUI"
gui.ResetOnSpawn = false
gui.Enabled = false
gui.Parent = playerGui

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(360, 440)
main.Position = UDim2.new(0.5, -180, 0.5, -220)
main.BackgroundColor3 = Color3.fromRGB(24, 24, 30)
main.BorderSizePixel = 0
main.Parent = gui

local function make(class, text, size, pos, parent)
    local obj = Instance.new(class)
    obj.Size = size
    obj.Position = pos
    obj.Parent = parent or main

    if obj:IsA("TextLabel") or obj:IsA("TextButton") then
        obj.Text = text
        obj.TextColor3 = Color3.new(1, 1, 1)
        obj.TextSize = 14
        obj.Font = Enum.Font.Gotham
    end

    return obj
end

make("TextLabel", "ZOMBIE CONTROL", UDim2.new(1, -20, 0, 30),
    UDim2.fromOffset(10, 5)).BackgroundTransparency = 1

local search = make("TextBox", "", UDim2.new(1, -20, 0, 30),
    UDim2.fromOffset(10, 40))
search.PlaceholderText = "Search zombie names..."
search.Text = ""
search.TextColor3 = Color3.new(1, 1, 1)
search.BackgroundColor3 = Color3.fromRGB(42, 42, 50)
search.ClearTextOnFocus = false

local list = make("ScrollingFrame", "", UDim2.new(1, -20, 0, 155),
    UDim2.fromOffset(10, 78))
list.BackgroundColor3 = Color3.fromRGB(32, 32, 38)
list.BorderSizePixel = 0
list.ScrollBarThickness = 5
list.AutomaticCanvasSize = Enum.AutomaticSize.Y
list.CanvasSize = UDim2.new()

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 3)
layout.Parent = list

local selectedLabel = make("TextLabel", "Selected: Zombie",
    UDim2.new(1, -20, 0, 24), UDim2.fromOffset(10, 240))
selectedLabel.BackgroundTransparency = 1

local sizeBox = make("TextBox", "2",
    UDim2.new(0.45, -10, 0, 30), UDim2.fromOffset(10, 268))
sizeBox.TextColor3 = Color3.new(1, 1, 1)
sizeBox.BackgroundColor3 = Color3.fromRGB(42, 42, 50)
sizeBox.ClearTextOnFocus = false

local apply = make("TextButton", "RESIZE HEAD",
    UDim2.new(0.55, -10, 0, 30), UDim2.new(0.45, 0, 0, 268))
apply.BackgroundColor3 = Color3.fromRGB(45, 95, 155)

local reset = make("TextButton", "RESET HEAD SIZE",
    UDim2.new(1, -20, 0, 30), UDim2.fromOffset(10, 305))
reset.BackgroundColor3 = Color3.fromRGB(100, 55, 55)

local espButton = make("TextButton", "ESP: OFF",
    UDim2.new(1, -20, 0, 30), UDim2.fromOffset(10, 342))
espButton.BackgroundColor3 = Color3.fromRGB(55, 90, 65)

local status = make("TextLabel", "Right Shift: open / close",
    UDim2.new(1, -20, 0, 24), UDim2.fromOffset(10, 380))
status.BackgroundTransparency = 1

local function getZombies(name)
    local results = {}

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model")
            and obj.Name == name
            and obj:FindFirstChildOfClass("Humanoid")
            and obj:FindFirstChild("Head") then
            table.insert(results, obj)
        end
    end

    return results
end

local function rebuildList()
    for _, child in ipairs(list:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end

    for _, name in ipairs(zombieNames) do
        if name:lower():find(search.Text:lower(), 1, true) then
            local button = make("TextButton", name,
                UDim2.new(1, -6, 0, 27), UDim2.new(), list)
            button.BackgroundColor3 = name == selected
                and Color3.fromRGB(55, 85, 130)
                or Color3.fromRGB(43, 43, 50)
            button.Activated:Connect(function()
                selected = name
                selectedLabel.Text = "Selected: " .. name
                rebuildList()
            end)
        end
    end
end

search:GetPropertyChangedSignal("Text"):Connect(rebuildList)
rebuildList()

apply.Activated:Connect(function()
    local n = tonumber(sizeBox.Text)
    if not n or n < 0.5 or n > 20 then
        status.Text = "Use a multiplier from 0.5 to 20"
        return
    end

    local count = 0

    for _, npc in ipairs(getZombies(selected)) do
        local head = npc:FindFirstChild("Head")
        if head and head:IsA("BasePart") then
            if not originalSizes[head] then
                originalSizes[head] = head.Size
            end

            head.Size = originalSizes[head] * n
            count += 1
        end
    end

    status.Text = "Resized " .. count .. " zombie(s) locally"
end)

reset.Activated:Connect(function()
    local count = 0

    for _, npc in ipairs(getZombies(selected)) do
        local head = npc:FindFirstChild("Head")
        if head and originalSizes[head] then
            head.Size = originalSizes[head]
            count += 1
        end
    end

    status.Text = "Reset " .. count .. " zombie(s)"
end)

local function clearESP()
    for _, obj in pairs(highlights) do
        obj:Destroy()
    end
    for _, obj in pairs(billboards) do
        obj:Destroy()
    end
    table.clear(highlights)
    table.clear(billboards)
end

local function updateESP()
    clearESP()
    if not espEnabled then return end

    for _, name in ipairs(zombieNames) do
        for _, npc in ipairs(getZombies(name)) do
            local head = npc:FindFirstChild("Head")
            local humanoid = npc:FindFirstChildOfClass("Humanoid")

            if head and humanoid then
                local h = Instance.new("Highlight")
                h.Adornee = npc
                h.FillTransparency = 0.8
                h.OutlineColor = Color3.fromRGB(255, 75, 75)
                h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                h.Parent = npc
                table.insert(highlights, h)

                local billboard = Instance.new("BillboardGui")
                billboard.Size = UDim2.fromOffset(150, 45)
                billboard.StudsOffset = Vector3.new(0, 2.5, 0)
                billboard.AlwaysOnTop = true
                billboard.Adornee = head
                billboard.Parent = playerGui
                table.insert(billboards, billboard)

                local label = Instance.new("TextLabel")
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.TextColor3 = Color3.new(1, 1, 1)
                label.TextStrokeTransparency = 0
                label.TextScaled = true
                label.Text = name .. "\nHP: "
                    .. math.floor(humanoid.Health)
                    .. "/" .. math.floor(humanoid.MaxHealth)
                label.Parent = billboard
            end
        end
    end
end

espButton.Activated:Connect(function()
    espEnabled = not espEnabled
    espButton.Text = espEnabled and "ESP: ON" or "ESP: OFF"
    updateESP()
end)

UIS.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.RightShift then
        gui.Enabled = not gui.Enabled
    end
end)

-- Keep health text and ESP synced as NPCs spawn or change.
task.spawn(function()
    while gui.Parent do
        if espEnabled then
            updateESP()
        end
        task.wait(2)
    end
end)
