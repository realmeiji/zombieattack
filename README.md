local HITBOX_SIZE = 10

-- Exact zombie names from your list (lowercase for matching)
local ZOMBIE_NAMES = {
	-- Base
	"zombie", "fast zombie", "explosive zombie", "slime", "sand monster",
	"ice zombie", "lava zombie", "void slime", "vampire", "ghost", "angel",
	"demon", "archangel", "beast", "crystal", "santa", "predator zombie",

	-- Tank
	"tank zombie", "tank fast zombie", "tank explosive zombie", "tank slime",
	"tank sand monster", "tank ice zombie", "tank lava zombie", "tank void slime",
	"tank vampire", "tank ghost", "tank angel", "tank demon", "tank archangel",
	"tank beast", "tank crystal", "tank santa", "tank alien", "tank monsters",

	-- Enraged
	"enraged zombie", "enraged fast zombie", "enraged explosive zombie",
	"enraged slime", "enraged sand monster", "enraged ice zombie",
	"enraged lava zombie", "enraged void slime", "enraged vampire",
	"enraged ghost", "enraged angel", "enraged demon", "enraged archangel",
	"enraged beast", "enraged crystal", "enraged santa", "enraged alien",

	-- Aliens
	"alien", "fast alien", "explosive alien",
	"tank alien", "tank fast alien", "tank explosive alien",
	"enraged alien", "enraged fast alien", "enraged explosive alien",

	-- Pumpkin
	"pumpkin zombie", "fast pumpkin zombie", "explosive pumpkin zombie",
	"ice pumpkin zombie", "lava pumpkin zombie", "tank pumpkin zombie",
	"tank fast pumpkin zombie", "tank explosive pumpkin zombie",
	"tank ice pumpkin zombie", "tank lava pumpkin zombie",
	"enraged pumpkin zombie", "enraged fast pumpkin zombie",
	"enraged explosive pumpkin zombie", "enraged ice pumpkin zombie",
	"enraged lava pumpkin zombie",

	-- Elf
	"elf", "fast elf", "explosive elf",
	"tank elf", "tank fast elf", "tank explosive elf",
	"enraged elf", "enraged fast elf", "enraged explosive elf",

	-- Bosses
	"mega tank", "king slime", "dark ghost", "demon overlord",
	"dragon beast", "alien leader",

	-- Groups
	"elves", "elf raiders", "wingmen", "aliens",
	"zombie horde", "tank monsters", "enraged monsters",
}

-- ESP colors by keyword (checked top to bottom, first match wins)
local COLOR_MAP = {
	{ "enraged",   Color3.fromRGB(255, 0, 0) },
	{ "lava",      Color3.fromRGB(255, 90, 0) },
	{ "ice",       Color3.fromRGB(120, 200, 255) },
	{ "void",      Color3.fromRGB(140, 0, 220) },
	{ "crystal",   Color3.fromRGB(180, 255, 255) },
	{ "sand",      Color3.fromRGB(230, 200, 120) },
	{ "explosive", Color3.fromRGB(255, 200, 0) },
	{ "fast",      Color3.fromRGB(0, 255, 180) },
	{ "tank",      Color3.fromRGB(120, 120, 120) },
	{ "pumpkin",   Color3.fromRGB(255, 130, 0) },
	{ "elf",       Color3.fromRGB(0, 255, 90) },
	{ "santa",     Color3.fromRGB(255, 40, 40) },
	{ "vampire",   Color3.fromRGB(150, 0, 60) },
	{ "ghost",     Color3.fromRGB(200, 200, 255) },
	{ "dark",      Color3.fromRGB(60, 60, 90) },
	{ "angel",     Color3.fromRGB(255, 255, 180) },
	{ "archangel", Color3.fromRGB(255, 230, 120) },
	{ "demon",     Color3.fromRGB(180, 0, 0) },
	{ "overlord",  Color3.fromRGB(120, 0, 180) },
	{ "beast",     Color3.fromRGB(120, 60, 20) },
	{ "dragon",    Color3.fromRGB(200, 40, 40) },
	{ "alien",     Color3.fromRGB(90, 255, 120) },
	{ "leader",    Color3.fromRGB(0, 255, 200) },
	{ "slime",     Color3.fromRGB(0, 220, 120) },
	{ "king",      Color3.fromRGB(255, 215, 0) },
	{ "zombie",    Color3.fromRGB(80, 160, 80) },
	{ "monster",   Color3.fromRGB(180, 80, 180) },
	{ "wingmen",   Color3.fromRGB(100, 200, 255) },
	{ "raiders",   Color3.fromRGB(255, 160, 80) },
	{ "horde",     Color3.fromRGB(100, 200, 100) },
}

local DEFAULT_COLOR = Color3.fromRGB(255, 60, 60)

-- ============ HELPERS ============
local nameLookup = {}
for _, n in ipairs(ZOMBIE_NAMES) do
	nameLookup[n] = true
end

local function isZombie(model)
	if not model or not model:IsA("Model") then return false end
	if not model:FindFirstChild("Head") then return false end

	local n = model.Name:lower()
	if nameLookup[n] then return true end

	-- also match "Zombie" variants where name has extra stuff (e.g. "Zombie (1)")
	for _, kw in ipairs(ZOMBIE_NAMES) do
		if n:find(kw, 1, true) then return true end
	end
	return false
end

local function colorFor(model)
	local n = model.Name:lower()
	for _, entry in ipairs(COLOR_MAP) do
		if n:find(entry[1], 1, true) then
			return entry[2]
		end
	end
	return DEFAULT_COLOR
end

-- ============ SETUP ============
local function setupZombie(model)
	if not isZombie(model) then return end

	local head = model:FindFirstChild("Head")
	if not head or not head:IsA("BasePart") then return end

	-- hitbox: head = 10x10x10
	head.Size = Vector3.new(HITBOX_SIZE, HITBOX_SIZE, HITBOX_SIZE)
	head.Massless = true
	head.CanCollide = false

	-- esp highlight
	if not model:FindFirstChild("ZombieESP") then
		local color = colorFor(model)

		local hl = Instance.new("Highlight")
		hl.Name = "ZombieESP"
		hl.Adornee = model
		hl.FillColor = color
		hl.FillTransparency = 0.55
		hl.OutlineColor = color
		hl.OutlineTransparency = 0
		hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
		hl.Parent = model
	end
end

-- ============ SCAN ============
local function scanAll()
	for _, inst in ipairs(workspace:GetDescendants()) do
		if inst:IsA("Model") then
			setupZombie(inst)
		end
	end
end

scanAll()

workspace.DescendantAdded:Connect(function(inst)
	if inst:IsA("Model") then
		task.wait(0.1)
		setupZombie(inst)
	elseif inst:IsA("BasePart") and inst.Name == "Head" then
		local model = inst.Parent
		if model and model:IsA("Model") then
			task.wait(0.1)
			setupZombie(model)
		end
	end
end)

-- rescan every few seconds as safety net
task.spawn(function()
	while true do
		task.wait(5)
		scanAll()
	end
end)

print("[ZombieHitboxESP] Loaded")
