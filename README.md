-- ZombieHubClient (LocalScript)
-- Location: StarterPlayer > StarterPlayerScripts
-- Self-contained test build — no CollectionService tags needed

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- ============ CONFIG ============
local CONFIG = {
	Developer = "YOUR_USERNAME_HERE",
	Version = "v1.0.0-test",
	Changelog = {
		"v1.0.0-test - LocalScript build, auto name detection",
		"v1.0.0      - Initial release",
	},
	HitboxSize = 10,
	ESPColor = Color3.fromRGB(255, 60, 60),
	ESPTextColor = Color3.fromRGB(255, 255, 255),
	Accent = Color3.fromRGB(140, 90, 255),
	Accent2 = Color3.fromRGB(90, 200, 255),
	BG = Color3.fromRGB(20, 20, 28),
	BG2 = Color3.fromRGB(30, 30, 42),
	Text = Color3.fromRGB(235, 235, 245),
	SubText = Color3.fromRGB(160, 160, 180),
}

-- ============ ZOMBIE NAME KEYWORDS ============
-- Any Model whose Name contains one of these (case-insensitive) is treated as a zombie.
local ZOMBIE_KEYWORDS = {
	"zombie", "slime", "vampire", "ghost", "angel", "demon", "archangel",
	"beast", "crystal", "santa", "predator", "alien", "monster", "pumpkin",
	"elf", "mega tank", "king slime", "dark ghost", "demon overlord",
	"dragon beast", "alien leader", "wingmen", "raiders",
}

local function isZombie(model)
	if not model:IsA("Model") then return false end
	if not model.PrimaryPart and not model:FindFirstChild("Head") then return false end
	local n = model.Name:lower()
	for _, kw in ipairs(ZOMBIE_KEYWORDS) do
		if n:find(kw, 1, true) then return true end
	end
	return false
end

-- ============ STATE ============
local state = {
	hitboxOn = false,
	espOn = false,
	originalSizes = {},
	highlights = {},
	billboards = {},
	espConnections = {},
}

-- ============ HELPERS ============
local function make(class, props, children)
	local obj = Instance.new(class)
	for k, v in pairs(props or {}) do obj[k] = v end
	for _, c in ipairs(children or {}) do c.Parent = obj end
	return obj
end

local function corner(r)
	return make("UICorner", { CornerRadius = UDim.new(0, r or 8) })
end

local function stroke(color, thickness, transparency)
	return make("UIStroke", {
		Color = color or CONFIG.Accent,
		Thickness = thickness or 1,
		Transparency = transparency or 0,
		ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
	})
end

local function padding(n)
	return make("UIPadding", {
		PaddingTop = UDim.new(0, n),
		PaddingBottom = UDim.new(0, n),
		PaddingLeft = UDim.new(0, n),
		PaddingRight = UDim.new(0, n),
	})
end

-- ============ BUILD GUI ============
local gui = make("ScreenGui", {
	Name = "ZombieHub",
	ResetOnSpawn = false,
	IgnoreGuiInset = true,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
	Parent = playerGui,
})

-- ==== Floating Toggle Button ====
local toggleBtn = make("TextButton", {
	Name = "Toggle",
	Size = UDim2.fromOffset(56, 56),
	Position = UDim2.new(0, 20, 0.5, -28),
	BackgroundColor3 = CONFIG.BG2,
	Text = "",
	AutoButtonColor = false,
	Parent = gui,
}, {
	corner(14),
	stroke(CONFIG.Accent, 1.5),
})

make("UIGradient", {
	Color = ColorSequence.new(CONFIG.Accent, CONFIG.Accent2),
	Rotation = 45,
	Parent = toggleBtn,
})

make("TextLabel", {
	Size = UDim2.fromScale(1, 1),
	BackgroundTransparency = 1,
	Text = "☠",
	TextColor3 = CONFIG.Text,
	TextScaled = true,
	Font = Enum.Font.GothamBold,
	Parent = toggleBtn,
}, { padding(14) })

-- ==== Main Frame ====
local main = make("Frame", {
	Name = "Main",
	Size = UDim2.fromOffset(560, 380),
	Position = UDim2.new(0.5, -280, 0.5, -190),
	BackgroundColor3 = CONFIG.BG,
	BorderSizePixel = 0,
	Visible = false,
	Parent = gui,
}, {
	corner(16),
	stroke(CONFIG.Accent, 1.5, 0.2),
})

make("UIGradient", {
	Color = ColorSequence.new(CONFIG.BG, Color3.fromRGB(35, 30, 55)),
	Rotation = 90,
	Parent = main,
})

-- Header
local header = make("Frame", {
	Name = "Header",
	Size = UDim2.new(1, 0, 0, 50),
	BackgroundColor3 = CONFIG.BG2,
	BorderSizePixel = 0,
	Parent = main,
}, { corner(16) })

make("Frame", {
	Size = UDim2.new(1, 0, 0, 16),
	Position = UDim2.new(0, 0, 1, -16),
	BackgroundColor3 = CONFIG.BG2,
	BorderSizePixel = 0,
	Parent = header,
})

make("TextLabel", {
	Size = UDim2.new(1, -100, 1, 0),
	Position = UDim2.new(0, 20, 0, 0),
	BackgroundTransparency = 1,
	Text = "☠  ZOMBIE HUB",
	TextColor3 = CONFIG.Text,
	Font = Enum.Font.GothamBold,
	TextSize = 20,
	TextXAlignment = Enum.TextXAlignment.Left,
	Parent = header,
})

local closeBtn = make("TextButton", {
	Size = UDim2.fromOffset(32, 32),
	Position = UDim2.new(1, -44, 0, 9),
	BackgroundColor3 = Color3.fromRGB(255, 70, 70),
	Text = "✕",
	TextColor3 = Color3.new(1, 1, 1),
	Font = Enum.Font.GothamBold,
	TextSize = 16,
	AutoButtonColor = false,
	Parent = header,
}, { corner(8) })

-- Tab bar
local tabBar = make("Frame", {
	Size = UDim2.new(1, -32, 0, 40),
	Position = UDim2.new(0, 16, 0, 62),
	BackgroundTransparency = 1,
	Parent = main,
})

make("UIListLayout", {
	FillDirection = Enum.FillDirection.Horizontal,
	Padding = UDim.new(0, 8),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = tabBar,
})

local content = make("Frame", {
	Size = UDim2.new(1, -32, 1, -120),
	Position = UDim2.new(0, 16, 0, 110),
	BackgroundTransparency = 1,
	Parent = main,
})

-- ============ TAB SYSTEM ============
local tabs = {}
local pages = {}

local function makeTab(name, order)
	local btn = make("TextButton", {
		Name = name .. "Tab",
		Size = UDim2.fromOffset(120, 40),
		BackgroundColor3 = CONFIG.BG2,
		Text = name,
		TextColor3 = CONFIG.SubText,
		Font = Enum.Font.GothamMedium,
		TextSize = 14,
		AutoButtonColor = false,
		LayoutOrder = order,
		Parent = tabBar,
	}, { corner(10), stroke(Color3.fromRGB(60, 60, 80), 1, 0.5) })

	local page = make("Frame", {
		Name = name .. "Page",
		Size = UDim2.fromScale(1, 1),
		BackgroundTransparency = 1,
		Visible = false,
		Parent = content,
	})

	tabs[name] = btn
	pages[name] = page
	return btn, page
end

local function selectTab(name)
	for n, p in pairs(pages) do p.Visible = (n == name) end
	for n, b in pairs(tabs) do
		local active = (n == name)
		TweenService:Create(b, TweenInfo.new(0.2), {
			BackgroundColor3 = active and CONFIG.Accent or CONFIG.BG2,
			TextColor3 = active and Color3.new(1, 1, 1) or CONFIG.SubText,
		}):Play()
	end
end

makeTab("Main", 1)
makeTab("ESP", 2)
makeTab("Info", 3)

-- ============ SECTION + BUTTON FACTORY ============
local function makeSection(parent, title, order)
	local sec = make("Frame", {
		Size = UDim2.new(1, 0, 0, 64),
		BackgroundColor3 = CONFIG.BG2,
		LayoutOrder = order,
		Parent = parent,
	}, { corner(12), stroke(Color3.fromRGB(60, 60, 80), 1, 0.6) })

	make("TextLabel", {
		Size = UDim2.new(1, -24, 0, 24),
		Position = UDim2.new(0, 16, 0, 8),
		BackgroundTransparency = 1,
		Text = title,
		TextColor3 = CONFIG.Text,
		Font = Enum.Font.GothamBold,
		TextSize = 14,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = sec,
	})

	local row = make("Frame", {
		Size = UDim2.new(1, -32, 0, 24),
		Position = UDim2.new(0, 16, 0, 34),
		BackgroundTransparency = 1,
		Parent = sec,
	})

	return sec, row
end

local function makeButton(text, parent, w, color, order)
	return make("TextButton", {
		Size = UDim2.fromOffset(w or 140, 32),
		BackgroundColor3 = color or CONFIG.Accent,
		Text = text,
		TextColor3 = Color3.new(1, 1, 1),
		Font = Enum.Font.GothamBold,
		TextSize = 13,
		AutoButtonColor = false,
		LayoutOrder = order or 1,
		Parent = parent,
	}, { corner(8) })
end

-- ============ MAIN PAGE ============
local mainPage = pages["Main"]
make("UIListLayout", {
	Padding = UDim.new(0, 10),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = mainPage,
})

local _, hitboxRow = makeSection(mainPage, "Hitbox Modifier", 1)
make("UIListLayout", {
	FillDirection = Enum.FillDirection.Horizontal,
	Padding = UDim.new(0, 8),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = hitboxRow,
})

local hitboxBtn = makeButton("Enable Hitbox (Head = " .. CONFIG.HitboxSize .. ")", hitboxRow, 240, CONFIG.Accent, 1)

local function setHitboxSize(size)
	for _, m in ipairs(workspace:GetDescendants()) do
		if isZombie(m) then
			local head = m:FindFirstChild("Head")
			if head and head:IsA("BasePart") then
				if not state.originalSizes[head] then
					state.originalSizes[head] = head.Size
				end
				head.Size = Vector3.new(size, size, size)
				head.Massless = true
				head.CanCollide = false
			end
		end
	end
end

local function restoreHitboxSize()
	for head, size in pairs(state.originalSizes) do
		if head and head.Parent then head.Size = size end
	end
	state.originalSizes = {}
end

hitboxBtn.MouseButton1Click:Connect(function()
	state.hitboxOn = not state.hitboxOn
	if state.hitboxOn then
		setHitboxSize(CONFIG.HitboxSize)
		hitboxBtn.Text = "Disable Hitbox"
		hitboxBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
	else
		restoreHitboxSize()
		hitboxBtn.Text = "Enable Hitbox (Head = " .. CONFIG.HitboxSize .. ")"
		hitboxBtn.BackgroundColor3 = CONFIG.Accent
	end
end)

-- keep resizing newly spawned zombies
task.spawn(function()
	while true do
		task.wait(1)
		if state.hitboxOn then setHitboxSize(CONFIG.HitboxSize) end
	end
end)

local _, toolsRow = makeSection(mainPage, "Quick Tools", 2)
make("UIListLayout", {
	FillDirection = Enum.FillDirection.Horizontal,
	Padding = UDim.new(0, 8),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = toolsRow,
})

makeButton("Refresh ESP", toolsRow, 140, Color3.fromRGB(70, 130, 200), 1).MouseButton1Click:Connect(function()
	if state.espOn then
		for _, c in ipairs(state.espConnections) do c:Disconnect() end
		state.espConnections = {}
		for _, h in pairs(state.highlights) do if h then h:Destroy() end end
		for _, b in pairs(state.billboards) do if b then b:Destroy() end end
		state.highlights, state.billboards = {}, {}
		for _, m in ipairs(workspace:GetDescendants()) do
			if isZombie(m) then
				-- re-add via ESP page function
			end
		end
	end
end)

makeButton("Teleport to Random", toolsRow, 160, Color3.fromRGB(120, 200, 120), 2).MouseButton1Click:Connect(function()
	local list = {}
	for _, m in ipairs(workspace:GetDescendants()) do
		if isZombie(m) and m.PrimaryPart then table.insert(list, m) end
	end
	if #list > 0 then
		local target = list[math.random(1, #list)]
		local char = player.Character
		if char and char:FindFirstChild("HumanoidRootPart") then
			char.HumanoidRootPart.CFrame = target.PrimaryPart.CFrame * CFrame.new(0, 5, 0)
		end
	end
end)

-- ============ ESP PAGE ============
local espPage = pages["ESP"]
make("UIListLayout", {
	Padding = UDim.new(0, 10),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = espPage,
})

local _, espRow = makeSection(espPage, "ESP Controls", 1)
make("UIListLayout", {
	FillDirection = Enum.FillDirection.Horizontal,
	Padding = UDim.new(0, 8),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = espRow,
})

local espBtn = makeButton("Enable ESP", espRow, 160, CONFIG.Accent2, 1)

local function addESP(model)
	if state.highlights[model] then return end
	local anchor = model.PrimaryPart or model:FindFirstChild("Head")
	if not anchor then return end

	local hl = Instance.new("Highlight")
	hl.Name = "ZombieESP"
	hl.Adornee = model
	hl.FillColor = CONFIG.ESPColor
	hl.FillTransparency = 0.6
	hl.OutlineColor = CONFIG.ESPColor
	hl.OutlineTransparency = 0
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Parent = model
	state.highlights[model] = hl

	local bb = Instance.new("BillboardGui")
	bb.Name = "ZombieESPName"
	bb.Adornee = anchor
	bb.Size = UDim2.fromOffset(200, 40)
	bb.StudsOffset = Vector3.new(0, 3, 0)
	bb.AlwaysOnTop = true
	bb.Parent = anchor

	make("TextLabel", {
		Size = UDim2.fromScale(1, 1),
		BackgroundTransparency = 1,
		Text = model.Name,
		TextColor3 = CONFIG.ESPTextColor,
		TextStrokeTransparency = 0,
		TextStrokeColor3 = Color3.new(0, 0, 0),
		Font = Enum.Font.GothamBold,
		TextScaled = true,
		Parent = bb,
	})

	state.billboards[model] = bb
end

local function clearESP()
	for _, h in pairs(state.highlights) do if h then h:Destroy() end end
	for _, b in pairs(state.billboards) do if b then b:Destroy() end end
	state.highlights, state.billboards = {}, {}
	for _, c in ipairs(state.espConnections) do c:Disconnect() end
	state.espConnections = {}
end

local function buildESP()
	clearESP()
	for _, m in ipairs(workspace:GetDescendants()) do
		if isZombie(m) then addESP(m) end
	end
end

espBtn.MouseButton1Click:Connect(function()
	state.espOn = not state.espOn
	if state.espOn then
		buildESP()
		espBtn.Text = "Disable ESP"
		espBtn.BackgroundColor3 = Color3.fromRGB(255, 80, 80)

		-- watch for new zombies via Workspace.DescendantAdded
		table.insert(state.espConnections, workspace.DescendantAdded:Connect(function(inst)
			if state.espOn and inst:IsA("Model") then
				task.wait(0.1)
				if isZombie(inst) then addESP(inst) end
			end
		end))
	else
		clearESP()
		espBtn.Text = "Enable ESP"
		espBtn.BackgroundColor3 = CONFIG.Accent2
	end
end)

-- ============ INFO PAGE ============
local infoPage = pages["Info"]
make("UIListLayout", {
	Padding = UDim.new(0, 10),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = infoPage,
})

local function makeInfoCard(title, bodyText, order, height)
	local card = make("Frame", {
		Size = UDim2.new(1, 0, 0, height or 100),
		BackgroundColor3 = CONFIG.BG2,
		LayoutOrder = order,
		Parent = infoPage,
	}, { corner(12), stroke(Color3.fromRGB(60, 60, 80), 1, 0.6), padding(12) })

	make("TextLabel", {
		Size = UDim2.new(1, 0, 0, 22),
		BackgroundTransparency = 1,
		Text = title,
		TextColor3 = CONFIG.Accent2,
		Font = Enum.Font.GothamBold,
		TextSize = 15,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = card,
	})

	make("TextLabel", {
		Size = UDim2.new(1, 0, 1, -28),
		Position = UDim2.new(0, 0, 0, 26),
		BackgroundTransparency = 1,
		Text = bodyText,
		TextColor3 = CONFIG.Text,
		Font = Enum.Font.Gotham,
		TextSize = 13,
		TextWrapped = true,
		TextYAlignment = Enum.TextYAlignment.Top,
		TextXAlignment = Enum.TextXAlignment.Left,
		Parent = card,
	})
	return card
end

makeInfoCard("👤 Developer", CONFIG.Developer .. "\nVersion: " .. CONFIG.Version, 1, 70)
makeInfoCard("📜 Changelog", table.concat(CONFIG.Changelog, "\n"), 2, 120)

-- Zombie list card
local ZOMBIE_LIST = {
	"Zombie","Fast Zombie","Explosive Zombie","Slime","Sand Monster","Ice Zombie","Lava Zombie",
	"Void Slime","Vampire","Ghost","Angel","Demon","Archangel","Beast","Crystal","Santa",
	"Predator Zombie","Alien","Fast Alien","Explosive Alien",
	"Tank Zombie","Tank Fast Zombie","Tank Explosive Zombie","Tank Slime","Tank Sand Monster",
	"Tank Ice Zombie","Tank Lava Zombie","Tank Void Slime","Tank Vampire","Tank Ghost",
	"Tank Angel","Tank Demon","Tank Archangel","Tank Beast","Tank Crystal","Tank Santa",
	"Tank Alien","Tank Monsters",
	"Enraged Zombie","Enraged Fast Zombie","Enraged Explosive Zombie","Enraged Slime",
	"Enraged Sand Monster","Enraged Ice Zombie","Enraged Lava Zombie","Enraged Void Slime",
	"Enraged Vampire","Enraged Ghost","Enraged Angel","Enraged Demon","Enraged Archangel",
	"Enraged Beast","Enraged Crystal","Enraged Santa","Enraged Alien",
	"Enraged Fast Alien","Enraged Explosive Alien",
	"Pumpkin Zombie","Fast Pumpkin Zombie","Explosive Pumpkin Zombie","Ice Pumpkin Zombie",
	"Lava Pumpkin Zombie","Tank Pumpkin Zombie","Tank Fast Pumpkin Zombie",
	"Tank Explosive Pumpkin Zombie","Tank Ice Pumpkin Zombie","Tank Lava Pumpkin Zombie",
	"Enraged Pumpkin Zombie","Enraged Fast Pumpkin Zombie","Enraged Explosive Pumpkin Zombie",
	"Enraged Ice Pumpkin Zombie","Enraged Lava Pumpkin Zombie",
	"Elf","Fast Elf","Explosive Elf","Tank Elf","Tank Fast Elf","Tank Explosive Elf",
	"Enraged Elf","Enraged Fast Elf","Enraged Explosive Elf",
	"Mega Tank","King Slime","Dark Ghost","Demon Overlord","Dragon Beast","Alien Leader",
	"Elves","Elf Raiders","Wingmen","Aliens","Zombie Horde","Tank Monsters","Enraged Monsters",
}

local listCard = make("Frame", {
	Size = UDim2.new(1, 0, 0, 260),
	BackgroundColor3 = CONFIG.BG2,
	LayoutOrder = 3,
	Parent = infoPage,
}, { corner(12), stroke(Color3.fromRGB(60, 60, 80), 1, 0.6) })

make("TextLabel", {
	Size = UDim2.new(1, -24, 0, 26),
	Position = UDim2.new(0, 12, 0, 8),
	BackgroundTransparency = 1,
	Text = "🧟 Zombie Types (" .. tostring(#ZOMBIE_LIST) .. ")",
	TextColor3 = CONFIG.Accent2,
	Font = Enum.Font.GothamBold,
	TextSize = 15,
	TextXAlignment = Enum.TextXAlignment.Left,
	Parent = listCard,
})

local scroll = make("ScrollingFrame", {
	Size = UDim2.new(1, -20, 1, -44),
	Position = UDim2.new(0, 10, 0, 38),
	BackgroundTransparency = 1,
	BorderSizePixel = 0,
	ScrollBarThickness = 4,
	ScrollBarImageColor3 = CONFIG.Accent,
	CanvasSize = UDim2.new(0, 0, 0, 0),
	Parent = listCard,
})

make("UIListLayout", {
	Padding = UDim.new(0, 2),
	SortOrder = Enum.SortOrder.LayoutOrder,
	Parent = scroll,
})

make("UIPadding", { PaddingRight = UDim.new(0, 8), Parent = scroll })

for i, name in ipairs(ZOMBIE_LIST) do
	make("TextLabel", {
		Size = UDim2.new(1, 0, 0, 20),
		BackgroundTransparency = 1,
		Text = "• " .. name,
		TextColor3 = CONFIG.Text,
		Font = Enum.Font.Gotham,
		TextSize = 12,
		TextXAlignment = Enum.TextXAlignment.Left,
		LayoutOrder = i,
		Parent = scroll,
	})
end

scroll.CanvasSize = UDim2.new(0, 0, 0, #ZOMBIE_LIST * 22 + 8)

-- ============ DRAGGING ============
local dragging, dragStart, startPos

header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = main.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		main.Position = UDim2.new(
			startPos.X.Scale, startPos.X.Offset + delta.X,
			startPos.Y.Scale, startPos.Y.Offset + delta.Y
		)
	end
end)

-- ============ TOGGLE ============
local function setOpen(open)
	main.Visible = open
	toggleBtn.Visible = not open
	if open then
		main.Size = UDim2.fromOffset(500, 340)
		TweenService:Create(main, TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
			Size = UDim2.fromOffset(560, 380),
		}):Play()
	end
end

toggleBtn.MouseButton1Click:Connect(function() setOpen(true) end)
closeBtn.MouseButton1Click:Connect(function() setOpen(false) end)

UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.RightShift then
		setOpen(not main.Visible)
	end
end)

selectTab("Main")
print("[ZombieHub] Loaded " .. CONFIG.Version)
