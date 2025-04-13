local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESP_ENABLED = true
local ESP_FOLDER = {}

-- Toggle ESP com tecla K
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
		ESP_ENABLED = not ESP_ENABLED
		for _, item in pairs(ESP_FOLDER) do
			for _, element in pairs(item) do
				if element then
					element.Visible = ESP_ENABLED
				end
			end
		end
	end
end)

-- Criar ESP para um jogador
local function createESP(player)
	if player == LocalPlayer then return end
	if ESP_FOLDER[player] then return end

	local box = Drawing.new("Square")
	box.Thickness = 1
	box.Color = Color3.new(0, 1, 0)
	box.Filled = false
	box.Visible = false

	local nameText = Drawing.new("Text")
	nameText.Color = Color3.new(1, 1, 1)
	nameText.Size = 16
	nameText.Center = true
	nameText.Outline = true
	nameText.Visible = false

	local distanceText = Drawing.new("Text")
	distanceText.Color = Color3.new(1, 1, 0)
	distanceText.Size = 14
	distanceText.Center = true
	distanceText.Outline = true
	distanceText.Visible = false

	local hpText = Drawing.new("Text")
	hpText.Color = Color3.new(1, 0, 0)
	hpText.Size = 14
	hpText.Center = true
	hpText.Outline = true
	hpText.Visible = false

	ESP_FOLDER[player] = {box = box, name = nameText, dist = distanceText, hp = hpText}

	RunService.RenderStepped:Connect(function()
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
			local humanoid = player.Character:FindFirstChild("Humanoid")
			local hrp = player.Character:FindFirstChild("HumanoidRootPart")
			local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

			local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
			local scale = math.clamp(1 / (distance * 0.03), 0.5, 2)
			local boxSize = Vector2.new(60 * scale, 100 * scale)

			box.Position = Vector2.new(pos.X - boxSize.X / 2, pos.Y - boxSize.Y / 2)
			box.Size = boxSize
			box.Visible = ESP_ENABLED and onScreen

			nameText.Position = Vector2.new(pos.X, pos.Y - boxSize.Y / 2 - 16)
			nameText.Text = player.Name
			nameText.Visible = ESP_ENABLED and onScreen

			distanceText.Position = Vector2.new(pos.X, pos.Y + boxSize.Y / 2 + 2)
			distanceText.Text = string.format("%.0f m", distance)
			distanceText.Visible = ESP_ENABLED and onScreen

			hpText.Position = Vector2.new(pos.X, pos.Y + boxSize.Y / 2 + 18)
			hpText.Text = "HP: " .. math.floor(humanoid.Health)
			hpText.Visible = ESP_ENABLED and onScreen
		else
			box.Visible = false
			nameText.Visible = false
			distanceText.Visible = false
			hpText.Visible = false
		end
	end)
end

-- Monitorar novos jogadores
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		wait(1)
		createESP(player)
	end)
end)

-- Adicionar ESP a jogadores já conectados
for _, player in pairs(Players:GetPlayers()) do
	if player ~= LocalPlayer then
		createESP(player)
	end
end
