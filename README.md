-- ========================================
-- AUTO COMPRAR NOOBINI LUSININI (Pasarela) - Steal a Brainrot
-- Por Grok: Detecta en conveyor, va y compra AUTO 24/7
-- ========================================

getgenv().AutoBuyNoobini = true  -- Toggle: true=ON

local NPC_NAME = "Noobini Lusinini"  -- ← CAMBIA SI ES DISTINTO (ej: "NoobiniLusinini")
local NPC_TAG = "Brainrot"  -- Fallback tag común (deja "" si no)

local BUY_DISTANCE = 25  -- Para pasarela
local PATH_AGENT_RADIUS = 4
local PATH_AGENT_HEIGHT = 6

-- Servicios
local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")
local CollectionService = game:GetService("CollectionService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

-- Encontrar NPC MÁS CERCA (el listo en pasarela)
local function findClosestNPC()
    local candidates = {}
    
    -- Por nombre
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name == NPC_NAME and obj:FindFirstChild("HumanoidRootPart") then
            table.insert(candidates, obj)
        end
    end
    
    -- Por tag (si hay)
    if NPC_TAG ~= "" then
        local tagged = CollectionService:GetTagged(NPC_TAG)
        for _, npc in ipairs(tagged) do
            if npc.Name:find("Noobini") and npc:FindFirstChild("HumanoidRootPart") then
                table.insert(candidates, npc)
            end
        end
    end
    
    local closest = nil
    local minDist = math.huge
    
    for _, npc in ipairs(candidates) do
        local hrp = npc:FindFirstChild("HumanoidRootPart")
        if hrp then
            local dist = (rootpart.Position - hrp.Position).Magnitude
            if dist < minDist then
                minDist = dist
                closest = npc
            end
        end
    end
    
    return closest
end

-- Pathfinding a NPC (mejorado para conveyor)
local function goToNPC(npc)
    local hrp = npc:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local npcPos = hrp.Position
    
    local path = PathfindingService:CreatePath({
        AgentRadius = PATH_AGENT_RADIUS,
        AgentHeight = PATH_AGENT_HEIGHT,
        AgentCanJump = true,
        WaypointSpacing = 3,
        Costs = {  -- Evita obstáculos comunes
            Water = 20,
            Danger = math.huge
        }
    })
    
    local success = pcall(function()
        path:ComputeAsync(rootpart.Position, npcPos)
    end)
    
    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for i, waypoint in ipairs(waypoints) do
            if not getgenv().AutoBuyNoobini then return end
            
            humanoid:MoveTo(waypoint.Position)
            if waypoint.Action == Enum.PathWaypointAction.Jump then
                humanoid.Jump = true
            end
            
            humanoid.MoveToFinished:Wait(3)  -- Timeout
        end
    else
        -- Directo si path falla
        humanoid:MoveTo(npcPos)
        humanoid.MoveToFinished:Wait(3)
    end
end

-- Comprar (ProximityPrompt optimizado)
local function buyFromNPC(npc)
    -- Busca en todo el modelo (a veces está en Head o parte)
    for _, obj in ipairs(npc:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and (obj.ActionText == "Comprar" or obj.ObjectText:find("Noobini")) then
            fireproximityprompt(obj)
            return true
        end
    end
    
    -- Fallback ClickDetector
    local cd = npc:FindFirstChildOfClass("ClickDetector")
    if cd then
        fireclickdetector(cd)
        return true
    end
    
    return false
end

-- LOOP PRINCIPAL: Auto-farm toda la noche
spawn(function()
    while true do
        if getgenv().AutoBuyNoobini then
            local npc = findClosestNPC()
            if npc then
                local hrp = npc:FindFirstChild("HumanoidRootPart")
                local dist = (rootpart.Position - hrp.Position).Magnitude
                
                print("🛒 Noobini a", math.floor(dist), "studs")  -- Debug
                
                if dist > BUY_DISTANCE then
                    goToNPC(npc)
                else
                    if buyFromNPC(npc) then
                        print("✅ COMPRADO Noobini Lusinini!")
                        wait(1)  -- Cooldown compra
                    end
                end
            else
                print("🔍 Buscando Noobini en pasarela...")
                wait(1)
            end
        end
        wait(0.3)  -- Rápido pero no spam
    end
end)

-- Respawn auto
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
end)

print("🚀 AUTO NOOBINI LUSININI ACTIVADO! Toda la noche farm.")
print("Para parar: getgenv().AutoBuyNoobini = false")
