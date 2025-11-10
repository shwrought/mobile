-- ========================================
-- AUTO INFINITO $6,700,000 (Velocidad DEFAULT + Retry 2x)
-- LOOP ETERNO: Detecta → Camina normal → Compra (intenta 2 veces) → Repite
-- ========================================

local TARGET_PRICE = "$250"
local BUY_DISTANCE = 15

-- Servicios
local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

-- NO TOCA VELOCIDAD: Default (16)

-- Encontrar $6.7M más cerca
local function findClosest6700k()
    local candidates = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and (obj.ObjectText:find(TARGET_PRICE) or obj.ObjectText:find("6,700,000") or obj.ObjectText:find("6700000")) then
            local model = obj:FindFirstAncestorOfClass("Model")
            if model and model:FindFirstChild("HumanoidRootPart") then
                table.insert(candidates, {npc = model, prompt = obj})
                print("🎯 Encontrado " .. obj.ObjectText)
            end
        end
    end
    
    local closest = nil
    local minDist = math.huge
    for _, cand in ipairs(candidates) do
        local hrp = cand.npc:FindFirstChild("HumanoidRootPart")
        if hrp then
            local dist = (rootpart.Position - hrp.Position).Magnitude
            if dist < minDist then
                minDist = dist
                closest = cand
            end
        end
    end
    return closest
end

-- Caminar NORMAL (pathfinding, velocidad default)
local function walkToNPC(npc)
    local hrp = npc:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local targetPos = hrp.Position + Vector3.new(0, 0, -3)
    
    local path = PathfindingService:CreatePath({
        AgentRadius = 3,
        AgentHeight = 6,
        AgentCanJump = true,
        WaypointSpacing = 4
    })
    
    pcall(function()
        path:ComputeAsync(rootpart.Position, targetPos)
        if path.Status == Enum.PathStatus.Success then
            local waypoints = path:GetWaypoints()
            for _, wp in ipairs(waypoints) do
                humanoid:MoveTo(wp.Position)
                if wp.Action == Enum.PathWaypointAction.Jump then
                    humanoid.Jump = true
                end
                local connection
                connection = humanoid.MoveToFinished:Connect(function(reached)
                    connection:Disconnect()
                end)
                wait(4)  -- Tiempo para default speed
            end
        end
    end)
end

-- COMPRA CON RETRY 2 VECES (hasta que funcione)
local function buyWithRetry(prompt)
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 50
    end)
    
    for i = 1, 2 do  -- Intenta 2 veces
        fireproximityprompt(prompt, 0)
        wait(0.3)  -- Pequeño delay entre intentos
        print("💥 Intento " .. i .. "/2 para " .. TARGET_PRICE)
    end
    
    print("✅ Compra completada/reintentada para " .. TARGET_PRICE)
end

-- LOOP INFINITO: Detecta → Camina → Compra (retry) → Repite
spawn(function()
    while true do
        local target = findClosest6700k()
        if target then
            local hrp = target.npc:FindFirstChild("HumanoidRootPart")
            local dist = (rootpart.Position - hrp.Position).Magnitude
            
            print("🚶 $6.7M detectado a " .. math.floor(dist) .. " studs")
            
            if dist > BUY_DISTANCE then
                print("→ Caminando normal...")
                walkToNPC(target.npc)
            end
            
            print("→ Comprando con retry...")
            buyWithRetry(target.prompt)
            
            wait(1)  -- Pequeño cooldown post-compra
        else
            print("🔍 Buscando $6.7M... (pasarela/dealer)")
            wait(0.5)
        end
        
        wait(0.2)  -- Loop rápido pero no spam
    end
end)

-- Respawn auto
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
end)

print("🔄 AUTO INFINITO $6,700,000 ACTIVADO! (Default speed + Retry 2x)")
print("→ Camina normal, compra cada vez que detecta, reintenta si falla.")
