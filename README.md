-- ========================================
-- AUTO $6,700,000 ANTI-LAG (SINGLE CLICK) - Steal a Brainrot Dealer
-- FIX: Solo 1 fireproximityprompt + HoldDuration=0 → SIN SPAM/LAG
-- ========================================

getgenv().AutoBuy6700k = true

local TARGET_PRICE = "$250"
local BUY_DISTANCE = 50
local SPEED = 200

-- Servicios
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

-- Anti-lag: Velocidad + NoClip
humanoid.WalkSpeed = SPEED
humanoid.JumpPower = 100

local function noclip()
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = false end
    end
end

-- INSTANT PROMPT: HoldDuration = 0 (¡CLAVE ANTI-LAG!)
local function makeInstant(prompt)
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 200
    end)
end

-- Encontrar $6.7M (pasarela/dealer)
local function findClosest6700k()
    local candidates = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and (obj.ObjectText:find(TARGET_PRICE) or obj.ObjectText:lower():find("6700000")) then
            makeInstant(obj)  -- INSTANT YA
            local model = obj:FindFirstAncestorOfClass("Model")
            if model and model:FindFirstChild("HumanoidRootPart") then
                table.insert(candidates, {npc = model, prompt = obj})
                print("🎯 ENCONTRADO " .. obj.ObjectText)
            end
        end
    end
    
    local closest = nil
    local minDist = math.huge
    for _, cand in ipairs(candidates) do
        local hrp = cand.npc.HumanoidRootPart
        local dist = (rootpart.Position - hrp.Position).Magnitude
        if dist < minDist then minDist = dist; closest = cand end
    end
    return closest
end

-- TP RÁPIDO
local function tpToNPC(npc)
    local hrp = npc.HumanoidRootPart
    rootpart.CFrame = hrp.CFrame * CFrame.new(0, 5, -5)
    wait(0.05)  -- Mínimo
end

-- COMPRA SINGLE-CLICK (¡SIN SPAM!)
local function buySingle(prompt)
    makeInstant(prompt)
    fireproximityprompt(prompt, 0)  -- 0 = hold instantáneo, 1 sola vez
    print("💥 COMPRADO " .. TARGET_PRICE .. " (SINGLE CLICK)")
end

-- LOOP ANTI-LAG (Heartbeat pero smart)
spawn(function()
    RunService.Heartbeat:Connect(function()
        if getgenv().AutoBuy6700k then
            noclip()
            local target = findClosest6700k()
            if target then
                local dist = (rootpart.Position - target.npc.HumanoidRootPart.Position).Magnitude
                if dist > 10 then
                    tpToNPC(target.npc)
                else
                    buySingle(target.prompt)
                    wait(1.5)  -- Cooldown ANTI-SPAM (importante!)
                end
            end
        end
    end)
end)

-- Auto-rejoin si no hay stock (Dealer restock)
spawn(function()
    while true do
        wait(45)
        if getgenv().AutoBuy6700k then
            local found = false
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA("ProximityPrompt") and obj.ObjectText:find(TARGET_PRICE) then
                    found = true; break
                end
            end
            if not found then
                print("🔄 Rejoin por restock...")
                TeleportService:Teleport(game.PlaceId, player)
            end
        end
    end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
    humanoid.WalkSpeed = SPEED
end)

print("⚡ AUTO $6.7M ANTI-LAG ACTIVADO! (Single Click)")
print("Para parar: getgenv().AutoBuy6700k = false")
print("💡 Si lag: Baja SPEED a 100")
