--// SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TextChatService = game:GetService("TextChatService")
local Debris = game:GetService("Debris")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")

--------------------------------------------------
--// COR DO PERSONAGEM
--------------------------------------------------
local bodyColor = Color3.new(1,1,1)
local bc = character:FindFirstChildOfClass("BodyColors")
if bc then
    bodyColor = bc.TorsoColor3
end

--------------------------------------------------
--// CONFIGURAÇÃO DA CORRENTE (BEM LONGA)
--------------------------------------------------
local BALL_COUNT = 28            -- MUITO MAIS BOLAS
local BALL_SIZE = 0.8
local LAST_BALL_SIZE = 1.15
local SPACING = 0.16
local BASE_OFFSET = Vector3.new(0, -1.4, -0.75)

local folder = Instance.new("Folder", workspace)
folder.Name = "AURA_LOCAL"

local balls = {}

for i = 1, BALL_COUNT do
    local ball = Instance.new("Part")
    ball.Shape = Enum.PartType.Ball
    ball.Anchored = true
    ball.CanCollide = false
    ball.CastShadow = false

    if i == BALL_COUNT then
        ball.Size = Vector3.new(LAST_BALL_SIZE, LAST_BALL_SIZE, LAST_BALL_SIZE)
        ball.Color = Color3.fromRGB(255,0,0)
        ball.Material = Enum.Material.Neon
    else
        ball.Size = Vector3.new(BALL_SIZE, BALL_SIZE, BALL_SIZE)
        ball.Color = bodyColor
        ball.Material = Enum.Material.SmoothPlastic
    end

    ball.Parent = folder
    balls[i] = ball
end

local redBall = balls[#balls]

--------------------------------------------------
--// QUADRADOS BRANCOS CAINDO E FICANDO NO CHÃO
--------------------------------------------------
task.spawn(function()
    while true do
        task.wait(0.06)

        if not redBall or not redBall.Parent then continue end

        local block = Instance.new("Part")
        block.Size = Vector3.new(0.3, 0.3, 0.3)
        block.Color = Color3.new(1,1,1)
        block.Material = Enum.Material.Neon
        block.Anchored = false
        block.CanCollide = true
        block.CastShadow = false

        block.CFrame =
            redBall.CFrame
            * CFrame.new(
                math.random(-6,6)/10,
                -0.2,
                math.random(-6,6)/10
            )

        block.Velocity = Vector3.new(
            math.random(-1,1),
            -math.random(8,12),
            math.random(-1,1)
        )

        block.Parent = workspace
        -- NÃO REMOVE → fica no chão
    end
end)

--------------------------------------------------
--// SEGUIR PLAYER
--------------------------------------------------
RunService.RenderStepped:Connect(function()
    if not hrp.Parent then return end

    local baseCF =
        hrp.CFrame
        * CFrame.new(BASE_OFFSET)
        * CFrame.Angles(0, math.rad(90), 0)

    for i, ball in ipairs(balls) do
        ball.CFrame = baseCF * CFrame.new((i-1)*SPACING, 0, 0)
    end
end)

--------------------------------------------------
--// CHAT REAL (TEXTCHAT + LEGACY) - SÓ NA SUA VISÃO
--------------------------------------------------
local mensagens = {
    "MUITA AURA.",
    "SIGMA DEMAIS",
    "BONITÃO",
    "AURA+TUFO"
}

local function sendFakeChat(name, msg)
    -- Novo chat
    if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
        TextChatService:DisplaySystemMessage(
            "<font color='#ffffff'><b>"..name..":</b> "..msg.."</font>"
        )
    else
        -- Chat antigo
        StarterGui:SetCore("ChatMakeSystemMessage", {
            Text = name..": "..msg,
            Color = Color3.new(1,1,1),
            Font = Enum.Font.GothamBold,
            TextSize = 18
        })
    end
end

task.spawn(function()
    task.wait(2)
    while true do
        task.wait(math.random(2,4))

        local plrs = Players:GetPlayers()
        local fakePlr = plrs[math.random(#plrs)]
        local text = mensagens[math.random(#mensagens)]

        sendFakeChat(fakePlr.Name, text)
    end
end)
