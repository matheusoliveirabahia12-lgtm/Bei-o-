-- 🔪 MURDER MYSTERY + DEBUG PANEL (MOBILE)
-- Painel: Beiçola do Luiz e Guilherme
-- Uso: jogo próprio / servidor privado

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ===============================
-- REMOTE EVENT
-- ===============================
local RolesEvent = Instance.new("RemoteEvent")
RolesEvent.Name = "RolesEvent"
RolesEvent.Parent = ReplicatedStorage

-- ===============================
-- SISTEMA DE ROLES
-- ===============================
local Roles = {}

local function assignRoles()
	local plrs = Players:GetPlayers()
	if #plrs < 3 then return end

	Roles = {}

	for _, p in ipairs(plrs) do
		Roles[p.UserId] = "Innocent"
	end

	local murderer = plrs[math.random(#plrs)]
	Roles[murderer.UserId] = "Murderer"

	local sheriff
	repeat
		sheriff = plrs[math.random(#plrs)]
	until sheriff ~= murderer

	Roles[sheriff.UserId] = "Sheriff"

	RolesEvent:FireAllClients(Roles)
end

Players.PlayerAdded:Connect(assignRoles)
Players.PlayerRemoving:Connect(assignRoles)

-- ===============================
-- CRIAR PAINEL MOBILE (LocalScript)
-- ===============================
local function createDebugPanel(player)
	local gui = Instance.new("ScreenGui")
	gui.Name = "DebugPanel"
	gui.ResetOnSpawn = false

	local localScript = Instance.new("LocalScript")
	localScript.Source = [[
		local Players = game:GetService("Players")
		local ReplicatedStorage = game:GetService("ReplicatedStorage")
		local player = Players.LocalPlayer

		local RolesEvent = ReplicatedStorage:WaitForChild("RolesEvent")

		local showRoles = false
		local showHitbox = false
		local rolesCache = {}

		-- ================= UI =================
		local frame = Instance.new("Frame")
		frame.Size = UDim2.new(0,260,0,280)
		frame.Position = UDim2.new(0,20,0.5,-140)
		frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
		frame.Parent = script.Parent

		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0,14)
		corner.Parent = frame

		local title = Instance.new("TextLabel")
		title.Size = UDim2.new(1,-10,0,60)
		title.Position = UDim2.new(0,5,0,5)
		title.BackgroundTransparency = 1
		title.TextWrapped = true
		title.Text = "Beiçola do Luiz\ne Guilherme"
		title.TextColor3 = Color3.new(1,1,1)
		title.Font = Enum.Font.GothamBold
		title.TextSize = 16
		title.Parent = frame

		local function createButton(text, y)
			local btn = Instance.new("TextButton")
			btn.Size = UDim2.new(1,-20,0,45)
			btn.Position = UDim2.new(0,10,0,y)
			btn.BackgroundColor3 = Color3.fromRGB(45,45,45)
			btn.TextColor3 = Color3.new(1,1,1)
			btn.Font = Enum.Font.Gotham
			btn.TextSize = 14
			btn.Text = text
			btn.Parent = frame

			local c = Instance.new("UICorner")
			c.CornerRadius = UDim.new(0,8)
			c.Parent = btn

			return btn
		end

		local btnRoles = createButton("Mostrar Murder / Sheriff", 80)
		local btnHitbox = createButton("Mostrar Hitbox", 135)

		-- ================= ROLES =================
		RolesEvent.OnClientEvent:Connect(function(roles)
			rolesCache = roles
			if showRoles then
				for _, plr in pairs(Players:GetPlayers()) do
					local role = rolesCache[plr.UserId]
					if role then
						warn(plr.Name .. " é " .. role)
					end
				end
			end
		end)

		-- ================= HITBOX =================
		local function updateHitbox()
			for _, plr in pairs(Players:GetPlayers()) do
				if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
					if showHitbox then
						if not plr.Character:FindFirstChild("DebugHitbox") then
							local box = Instance.new("BoxHandleAdornment")
							box.Name = "DebugHitbox"
							box.Adornee = plr.Character.HumanoidRootPart
							box.Size = Vector3.new(4,6,4)
							box.Color3 = Color3.fromRGB(255,0,0)
							box.Transparency = 0.5
							box.AlwaysOnTop = true
							box.Parent = plr.Character
						end
					else
						local box = plr.Character:FindFirstChild("DebugHitbox")
						if box then box:Destroy() end
					end
				end
			end
		end

		-- ================= BOTÕES =================
		btnRoles.MouseButton1Click:Connect(function()
			showRoles = not showRoles
			btnRoles.Text = showRoles and "Ocultar Murder / Sheriff" or "Mostrar Murder / Sheriff"

			if showRoles then
				for _, plr in pairs(Players:GetPlayers()) do
					local role = rolesCache[plr.UserId]
					if role then
						warn(plr.Name .. " é " .. role)
					end
				end
			end
		end)

		btnHitbox.MouseButton1Click:Connect(function()
			showHitbox = not showHitbox
			btnHitbox.Text = showHitbox and "Ocultar Hitbox" or "Mostrar Hitbox"
			updateHitbox()
		end)
	]]

	localScript.Parent = gui
	gui.Parent = player:WaitForChild("PlayerGui")
end

Players.PlayerAdded:Connect(createDebugPanel)
