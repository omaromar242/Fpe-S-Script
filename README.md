-- Load the DrRay UI Library
local DrRayLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/AZYsGithub/DrRay-UI-Library/main/DrRay.lua"))()

-- Create the UI window
local window = DrRayLibrary:Load("DrRay", "Default")

-- Create a new tab named "Fpe:S"
local tab = DrRayLibrary.newTab("Fpe:S", "ImageIdHere")

-- Services and local player reference
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

------------------------------------------------
-- ESP Students/Teachers Feature Implementation
------------------------------------------------
local ESPEnabled = false

-- Function to add ESP outline to a player's character
local function applyESP(player)
	if player == LocalPlayer then return end
	local character = player.Character
	if character and not character:FindFirstChild("FpeHighlight") then
		local highlight = Instance.new("Highlight")
		highlight.Name = "FpeHighlight"
		highlight.Adornee = character
		highlight.FillTransparency = 1       -- Fully transparent fill (outline-only)
		highlight.OutlineTransparency = 0    -- Fully visible outline
		-- Use player's team color (fallback to white if not available)
		highlight.OutlineColor = (player.TeamColor and player.TeamColor.Color) or Color3.new(1,1,1)
		highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- See through walls
		highlight.Parent = character
	end
end

-- Function to remove ESP outline from a player's character
local function removeESP(player)
	local character = player.Character
	if character then
		local h = character:FindFirstChild("FpeHighlight")
		if h then
			h:Destroy()
		end
	end
end

-- Update ESP for all players
local function updateAllESP(enabled)
	for _, player in pairs(Players:GetPlayers()) do
		if enabled then
			applyESP(player)
		else
			removeESP(player)
		end
	end
end

-- Connect CharacterAdded to reapply ESP on respawn
local function connectCharacter(player)
	player.CharacterAdded:Connect(function(character)
		if ESPEnabled then
			wait(0.5)
			applyESP(player)
		end
	end)
end

-- Connect for all current and future players
for _, player in pairs(Players:GetPlayers()) do
	connectCharacter(player)
end
Players.PlayerAdded:Connect(connectCharacter)

-- Add the ESP toggle to the tab
tab.newToggle("ESP Students/Teachers", "Toggle outline-only ESP with team color (see through walls)", false, function(state)
	ESPEnabled = state
	updateAllESP(ESPEnabled)
	print("[ESP] Students/Teachers is now:", state)
end)

------------------------------------------------
-- Hitbox All Feature Implementation
------------------------------------------------
local HitboxEnabled = false
local hitboxLoopActive = false

-- Function to update hitboxes for all non-local players
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

-- Add the Hitbox All toggle to the tab
tab.newToggle("Hitbox All", "Every 2 seconds, sets all other players' HumanoidRootPart size to (10,10,10)", false, function(state)
	HitboxEnabled = state
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
	print("[Hitbox All] is now:", state)
end)

------------------------------------------------
-- (Optional) Built-in UI functions from DrRay:
-- You can toggle/hide the UI using:
-- window:Toggle(), window:Open(), window:Close(), window:Hide(), window:Show()
------------------------------------------------

-- End of script
