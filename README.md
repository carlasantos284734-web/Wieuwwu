-- Script de Teste - Protótipo estilo Blox Fruits com Auto Farm
-- Coloque em StarterPlayer > StarterPlayerScripts

local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local HRP = char:WaitForChild("HumanoidRootPart")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Cooldowns
local cooldowns = {
    Dash = false,
    Power1 = false,
    Power2 = false,
    Power3 = false,
}

-- ======== FUNÇÕES DE PODER =========

local function Dash()
    if cooldowns.Dash then return end
    cooldowns.Dash = true

    HRP.Velocity = HRP.CFrame.LookVector * 100

    wait(2)
    cooldowns.Dash = false
end

local function Power1()
    if cooldowns.Power1 then return end
    cooldowns.Power1 = true

    local proj = Instance.new("Part")
    proj.Shape = Enum.PartType.Ball
    proj.Color = Color3.fromRGB(0, 200, 255)
    proj.Material = Enum.Material.Neon
    proj.Size = Vector3.new(2,2,2)
    proj.CFrame = HRP.CFrame * CFrame.new(0,0,-5)
    proj.Anchored = false
    proj.CanCollide = false
    proj.Parent = workspace

    local bv = Instance.new("BodyVelocity")
    bv.Velocity = HRP.CFrame.LookVector * 150
    bv.MaxForce = Vector3.new(1e5,1e5,1e5)
    bv.Parent = proj

    game.Debris:AddItem(proj, 5)

    wait(2)
    cooldowns.Power1 = false
end

local function Power2()
    if cooldowns.Power2 then return end
    cooldowns.Power2 = true

    local explosion = Instance.new("Explosion")
    explosion.Position = HRP.Position
    explosion.BlastRadius = 10
    explosion.BlastPressure = 50000
    explosion.Parent = workspace

    wait(5)
    cooldowns.Power2 = false
end

local flying = false
local function Power3()
    if cooldowns.Power3 then return end
    cooldowns.Power3 = true

    flying = not flying
    if flying then
        humanoid.PlatformStand = true
        while flying do
            task.wait()
            HRP.Velocity = HRP.CFrame.LookVector * 80
        end
    else
        humanoid.PlatformStand = false
    end

    wait(1)
    cooldowns.Power3 = false
end

-- ======== AUTO FARM =========

local autoFarm = false

local function getNearestEnemy()
    local closest, dist = nil, math.huge
    for _, npc in pairs(workspace:GetChildren()) do
        if npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") and npc.Name ~= player.Name then
            local d = (npc.HumanoidRootPart.Position - HRP.Position).Magnitude
            if d < dist and npc.Humanoid.Health > 0 then
                closest = npc
                dist = d
            end
        end
    end
    return closest
end

local function startAutoFarm()
    while autoFarm do
        task.wait(0.5)
        local enemy = getNearestEnemy()
        if enemy then
            -- mover até o inimigo
            HRP.CFrame = CFrame.new(HRP.Position, enemy.HumanoidRootPart.Position)
            humanoid:MoveTo(enemy.HumanoidRootPart.Position)

            -- atacar
            Power1()
        end
    end
end

-- ======== TECLAS =========

UIS.InputBegan:Connect(function(input, isTyping)
    if isTyping then return end
    if input.KeyCode == Enum.KeyCode.Q then
        Dash()
    elseif input.KeyCode == Enum.KeyCode.Z then
        Power1()
    elseif input.KeyCode == Enum.KeyCode.X then
        Power2()
    elseif input.KeyCode == Enum.KeyCode.C then
        Power3()
    elseif input.KeyCode == Enum.KeyCode.F then
        autoFarm = not autoFarm
        if autoFarm then
            print("Auto Farm ON")
            task.spawn(startAutoFarm)
        else
            print("Auto Farm OFF")
        end
    end
end)
