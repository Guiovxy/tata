local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local camera = workspace.CurrentCamera


local frame = script.Parent
local headBtn = frame:WaitForChild("HeadToggleButton")
local teleportBtn = frame:WaitForChild("TeleportToggleButton")
local auraBtn = frame:WaitForChild("AuraToggleButton")
local voidBtn = frame:WaitForChild("VoidButton")
local targetBox = frame:WaitForChild("TargetBox")
local targetLabel = frame:WaitForChild("TargetLabel")
local invincibilityBtn = frame:WaitForChild("InvincibilityButton")


local headLocked = false
local teleportEnabled = false
local auraEnabled = false
local selectedTarget = nil
local isInvincible = false


local function toggleHead()
	local character = player.Character or player.CharacterAdded:Wait()
	local humanoid = character:WaitForChild("Humanoid")
	local neck = humanoid:FindFirstChild("Neck") or character:FindFirstChild("UpperTorso"):FindFirstChild("Neck")

	if neck then
		headLocked = not headLocked
		if headLocked then
			neck.C0 = CFrame.new(0, 1, 0)
			neck.C1 = CFrame.new(0, -0.5, 0)
		else
			neck.C0 = CFrame.new(0, 1, 0)
			neck.C1 = CFrame.new(0, -0.5, 0)
		end
	end
end


local function teleportToLook()
	local character = player.Character or player.CharacterAdded:Wait()
	local hrp = character:WaitForChild("HumanoidRootPart")

	local direction = camera.CFrame.LookVector * 200
	local origin = camera.CFrame.Position

	local rayParams = RaycastParams.new()
	rayParams.FilterDescendantsInstances = {character}
	rayParams.FilterType = Enum.RaycastFilterType.Blacklist

	local result = workspace:Raycast(origin, direction, rayParams)

	if result then
		local pos = result.Position + Vector3.new(0, 3, 0)
		hrp.CFrame = CFrame.new(pos)
	end
end


targetBox.FocusLost:Connect(function()
	local name = targetBox.Text
	if name and name ~= "" then
		selectedTarget = name
		targetLabel.Text = "Jogador alvo: " .. name
	end
end)


local function toggleInvincibility()
	if isInvincible then
		
		isInvincible = false
		invincibilityBtn.Text = "Ativar Intocável"
		for _, part in ipairs(character:GetChildren()) do
			if part:IsA("BasePart") then
				
				part.CanCollide = true
			end
		end
	else
		
		isInvincible = true
		invincibilityBtn.Text = "Desativar Intocável"
		for _, part in ipairs(character:GetChildren()) do
			if part:IsA("BasePart") then
				
				part.CanCollide = false
			end
		end
	end
end


headBtn.MouseButton1Click:Connect(toggleHead)

teleportBtn.MouseButton1Click:Connect(function()
	teleportEnabled = not teleportEnabled
	teleportBtn.Text = teleportEnabled and "Desativar Teleporte" or "Ativar Teleporte"
end)

auraBtn.MouseButton1Click:Connect(function()
	auraEnabled = not auraEnabled
	auraBtn.Text = auraEnabled and "Desativar Aura" or "Ativar Aura"
	ReplicatedStorage:WaitForChild("ToggleAura"):FireServer(auraEnabled)
end)

voidBtn.MouseButton1Click:Connect(function()
	if selectedTarget then
		ReplicatedStorage:WaitForChild("ToggleVoidLoop"):FireServer(selectedTarget)
	end
end)

invincibilityBtn.MouseButton1Click:Connect(toggleInvincibility)

UserInputService.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Enum.KeyCode.E then
		toggleHead()
	elseif input.KeyCode == Enum.KeyCode.Z and teleportEnabled then
		teleportToLook()
	end
end)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local auraEvent = ReplicatedStorage:WaitForChild("ToggleAura")
local voidEvent = ReplicatedStorage:WaitForChild("ToggleVoidLoop")

local auraStatus = {}
local voidTargets = {}

auraEvent.OnServerEvent:Connect(function(player, enabled)
	auraStatus[player] = enabled
end)

voidEvent.OnServerEvent:Connect(function(sender, targetName)
	local target = Players:FindFirstChild(targetName)
	if target then
		voidTargets[target] = true
	end
end)

while true do
	
	for player, enabled in pairs(auraStatus) do
		if enabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local origin = player.Character.HumanoidRootPart.Position
			for _, other in ipairs(Players:GetPlayers()) do
				if other ~= player and other.Character and other.Character:FindFirstChild("HumanoidRootPart") then
					local otherHRP = other.Character.HumanoidRootPart
					if (origin - otherHRP.Position).Magnitude < 10 then
						local bv = Instance.new("BodyVelocity")
						bv.Velocity = (otherHRP.Position - origin).Unit * 100
						bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
						bv.Parent = otherHRP
						game.Debris:AddItem(bv, 0.2)
					end
				end
			end
		end
	end

	
	for target, active in pairs(voidTargets) do
		if active and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
			target.Character.HumanoidRootPart.CFrame = CFrame.new(0, -1000, 0)
		end
	end

	wait(1)
end
