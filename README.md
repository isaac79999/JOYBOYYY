--// Aura Haki Joy Boy (VISUAL ONLY)
--// Executar no Delta | Client-side

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local lp = Players.LocalPlayer
local char = lp.Character or lp.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local humanoid = char:WaitForChild("Humanoid")

local auraEnabled = false
local auraParts = {}
local affected = {}

--// CONFIG
local AURA_RADIUS = 25
local PULSE_SPEED = 2

--// Criar Aura
local function createAura()
	for i = 1, 6 do
		local p = Instance.new("Part")
		p.Size = Vector3.new(1,1,1)
		p.Shape = Enum.PartType.Ball
		p.Anchored = true
		p.CanCollide = false
		p.Material = Enum.Material.Neon
		p.Color = Color3.fromRGB(30,30,30)
		p.Transparency = 0.35
		p.Parent = workspace

		local smoke = Instance.new("ParticleEmitter", p)
		smoke.Texture = "rbxassetid://7712212246"
		smoke.Rate = 60
		smoke.Lifetime = NumberRange.new(0.6)
		smoke.Speed = NumberRange.new(2)
		smoke.Size = NumberSequence.new{
			NumberSequenceKeypoint.new(0,2),
			NumberSequenceKeypoint.new(1,4)
		}

		table.insert(auraParts, p)
	end
end

--// Remover Aura
local function removeAura()
	for _,p in pairs(auraParts) do
		p:Destroy()
	end
	auraParts = {}

	for char,data in pairs(affected) do
		if char and char:FindFirstChild("HumanoidRootPart") then
			for _,part in pairs(char:GetDescendants()) do
				if part:IsA("BasePart") then
					part.Color = data.Color
					part.Anchored = false
				end
			end
		end
	end
	affected = {}
end

--// Efeito nos players (VISUAL)
local function affectPlayers()
	for _,plr in pairs(Players:GetPlayers()) do
		if plr ~= lp and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
			local chr = plr.Character
			local dist = (chr.HumanoidRootPart.Position - hrp.Position).Magnitude

			if dist <= AURA_RADIUS and not affected[chr] then
				affected[chr] = {Color = chr.HumanoidRootPart.Color}

				for _,part in pairs(chr:GetDescendants()) do
					if part:IsA("BasePart") then
						part.Color = Color3.fromRGB(90,90,90)
						part.Anchored = true
					end
				end
			end
		end
	end
end

--// Atualização Aura
RunService.RenderStepped:Connect(function(t)
	if not auraEnabled then return end

	for i,p in pairs(auraParts) do
		local angle = t * (PULSE_SPEED + i)
		local radius = 4 + math.sin(t * 2)
		p.Position = hrp.Position + Vector3.new(
			math.cos(angle) * radius,
			1.5,
			math.sin(angle) * radius
		)
	end

	affectPlayers()
end)

--// Toggle Tecla H
UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Enum.KeyCode.H then
		auraEnabled = not auraEnabled

		if auraEnabled then
			createAura()
		else
			removeAura()
		end
	end
end)

--// Atualizar Character após morte
lp.CharacterAdded:Connect(function(c)
	char = c
	hrp = c:WaitForChild("HumanoidRootPart")
	humanoid = c:WaitForChild("Humanoid")
end)
