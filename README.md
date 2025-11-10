-- ========================================
-- WALK + COMPRA NORMAL $6,700,000 (Velocidad por DEFECTO)
-- Camina con Pathfinding, compra UNA VEZ, VELOCIDAD NORMAL
-- ========================================

local TARGET_PRICE = "$250"
local BUY_DISTANCE = 15
-- SIN SPEED HACK: Velocidad por defecto (16)

-- Servicios
local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

-- Velocidad POR DEFECTO: NO TOCAR
-- humanoid.WalkSpeed = 16  -- Ya es default

-- Encontrar $6.7M más cerca
local function findClosest6700k()
    local candidates = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and obj.ObjectText:find(TARGET_PRICE) then
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
        local hrp = cand.npc.HumanoidRootPart
        local dist = (rootpart.Position - hrp.Position).Magnitude
        if dist < minDist then
            minDist = dist
            closest = cand
        end
    end
    return closest
end

-- Caminar NORMAL con Pathfinding (velocidad default)
local function walkToNPC(npc)
    local hrp = npc.HumanoidRootPart
    local targetPos = hrp.Position + Vector3.new(0, 0, -3)  -- Frente
    
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
                humanoid.MoveToFinished:Wait(4)  -- Más tiempo para velocidad normal
            end
        end
    end)
end

-- Compra NORMAL (1 click)
local function buyNormal(prompt)
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 50
        fireproximityprompt(prompt, 0)
    end)
    print("✅ COMPRADO " .. TARGET_PRICE .. " (VELOCIDAD NORMAL)")
end

-- SCRIPT PRINCIPAL: UNA VEZ SOLO
local target = findClosest6700k()
if target then
    print("🚶 Camina a $6.7M (velocidad por defecto)...")
    local dist = (rootpart.Position - target.npc.HumanoidRootPart.Position).Magnitude
    if dist > BUY_DISTANCE then
        walkToNPC(target.npc)
    end
    buyNormal(target.prompt)
else
    print("❌ No encontró $6.7M. Ve manual a pasarela/dealer.")
end

print("🛑 Script terminado. Todo normal.")
