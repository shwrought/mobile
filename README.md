-- ========================================
-- AUTO $6,700,000 POST-DEALER UPDATE (Anti-Patch)
-- Bypass fireproximityprompt + TP + NoClip → COMPRA YA
-- ========================================

getgenv().AutoBuy6700k = true

local TARGET_PRICE = "$250"  -- ← Tu precio
local BUY_DISTANCE = 50  -- Mayor para dealer
local SPEED = 200
local FLY_SPEED = 100

-- Servicios
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

-- NoClip + Speed
local function noclip()
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end

humanoid.WalkSpeed = SPEED
humanoid.JumpPower = 100

-- Encontrar CUALQUIER $6,700,000 (pasarela/dealer)
local function findClosest6700k()
    local candidates = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            if obj.ObjectText:find(TARGET_PRICE) or obj.ObjectText:lower():find("6700000") then
                local model = obj:FindFirstAncestorOfClass("Model")
                if model and model:FindFirstChild("HumanoidRootPart") then
                    table.insert(candidates, {npc = model, prompt = obj})
                    print("🎯 ENCONTRADO " .. TARGET_PRICE .. ": " .. obj.ObjectText .. " | Action: " .. obj.ActionText)
                end
            end
        end
    end
    
    local closest = nil
    local minDist = math.huge
    for _, cand in ipairs(candidates) do
        local hrp = cand.npc.HumanoidRootPart
        local dist = (rootpart.Position - hrp.Position).Magnitude
        if dist < minDist then
            minDist = dist
            closest = cand
        end
    end
    return closest
end

-- Teleport INSTANTANEO (mejor que path)
local function tpToNPC(npc)
    local hrp = npc.HumanoidRootPart
    rootpart.CFrame = hrp.CFrame * CFrame.new(0, 5, -5)  -- Frente al NPC
    wait(0.1)
end

-- BYPASS COMPRA (nuevo método anti-patch)
local function buyBypass(prompt)
    pcall(function()
        prompt.MaxActivationDistance = 100  -- Fuerza activación
        prompt:InputHoldBegin()
        wait(0.1)
        fireproximityprompt(prompt, 0)  -- Fallback
        prompt:InputHoldEnd()
    end)
    print("💥 BYPASS ACTIVADO en " .. prompt.ObjectText)
end

-- LOOP ULTRA-RÁPIDO
spawn(function()
    RunService.Heartbeat:Connect(function()
        if getgenv().AutoBuy6700k then
            noclip()  -- Siempre NoClip
            
            local target = findClosest6700k()
            if target then
                local dist = (rootpart.Position - target.npc.HumanoidRootPart.Position).Magnitude
                print("🚀 $6.7M a " .. math.floor(dist) .. " | TP...")
                
                if dist > 10 then
                    tpToNPC(target.npc)
                else
                    buyBypass(target.prompt)
                    wait(1)
                end
            end
        end
    end)
end)

-- Auto-rejoin si no encuentra nada (servidor malo)
spawn(function()
    while true do
        wait(30)
        if getgenv().AutoBuy6700k then
            local found = false
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA("ProximityPrompt") and obj.ObjectText:find(TARGET_PRICE) then
                    found = true break
                end
            end
            if not found then
                print("🔄 Rejoin servidor nuevo...")
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

print("🔥 AUTO $6,700,000 ANTI-PATCH ACTIVADO!")
print("Mira consola: Si ve 'ENCONTRADO' → COMPRA.")
print("Toggle: getgenv().AutoBuy6700k = false")
