-- ========================================
-- AUTO BUY NPC SCRIPT por Grok
-- Detecta NPC por nombre/tag, va hacia él y compra en loop
-- Compatible con executors (fireproximityprompt, fireclickdetector)
-- ========================================

getgenv().AutoBuyNPC = true  -- Toggle: true = ON, false = OFF

local NPC_NAME = Noobini Lusinini  -- Ej: "ShopKeeper" (deja nil si usas tag)
local NPC_TAG = "Comun"  -- Etiqueta del NPC (ej: "Shop", "Vendor")

local BUY_DISTANCE = 15  -- Distancia para comprar (ajusta)
local PATH_AGENT_RADIUS = 3
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

-- Función para encontrar el NPC MÁS CERCA (por nombre o tag)
local function findClosestNPC()
    local candidates = {}
    
    if NPC_NAME then
        local npc = workspace:FindFirstChild(NPC_NAME, true)
        if npc then
            table.insert(candidates, npc)
        end
    else
        local tagged = CollectionService:GetTagged(NPC_TAG)
        for _, npc in ipairs(tagged) do
            if npc:IsA("Model") and (npc.PrimaryPart or npc:FindFirstChild("HumanoidRootPart")) then
                table.insert(candidates, npc)
            end
        end
    end
    
    local closest = nil
    local minDist = math.huge
    
    for _, npc in ipairs(candidates) do
        local npcPos = npc.PrimaryPart and npc.PrimaryPart.Position or npc:FindFirstChild("HumanoidRootPart") and npc.HumanoidRootPart.Position or npc:GetModelCFrame().Position
        local dist = (rootpart.Position - npcPos).Magnitude
        if dist < minDist then
            minDist = dist
            closest = npc
        end
    end
    
    return closest
end

-- Función para ir al NPC (Pathfinding)
local function goToNPC(npc)
    local npcPos = npc.PrimaryPart and npc.PrimaryPart.Position or npc:FindFirstChild("HumanoidRootPart") and npc.HumanoidRootPart.Position or npc:GetModelCFrame().Position
    
    local path = PathfindingService:CreatePath({
        AgentRadius = PATH_AGENT_RADIUS,
        AgentHeight = PATH_AGENT_HEIGHT,
        AgentCanJump = true,
        WaypointSpacing = 4
    })
    
    local success, err = pcall(function()
        path:ComputeAsync(rootpart.Position, npcPos)
    end)
    
    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for _, waypoint in ipairs(waypoints) do
            if getgenv().AutoBuyNPC == false then return end  -- Toggle check
            
            humanoid:MoveTo(waypoint.Position)
            if waypoint.Action == Enum.PathWaypointAction.Jump then
                humanoid.Jump = true
            end
            
            local reached = humanoid.MoveToFinished:Wait(2.5)  -- Timeout por waypoint
            if not reached then break end
        end
    else
        -- Fallback: Move directo si path falla
        humanoid:MoveTo(npcPos)
    end
end

-- Función para comprar
local function buyFromNPC(npc)
    local prompt = npc:FindFirstChildOfClass("ProximityPrompt")
    if prompt then
        fireproximityprompt(prompt)  -- Principal: ProximityPrompt
        return true
    end
    
    local cd = npc:FindFirstChildOfClass("ClickDetector")
    if cd then
        fireclickdetector(cd)  -- Fallback: ClickDetector
        return true
    end
    
    return false
end

-- Loop principal
spawn(function()
    while true do
        if getgenv().AutoBuyNPC then
            local npc = findClosestNPC()
            if npc then
                local npcPos = npc.PrimaryPart and npc.PrimaryPart.Position or npc:FindFirstChild("HumanoidRootPart") and npc.HumanoidRootPart.Position or npc:GetModelCFrame().Position
                local dist = (rootpart.Position - npcPos).Magnitude
                
                if dist > BUY_DISTANCE then
                    -- Ir al NPC
                    goToNPC(npc)
                else
                    -- Ya cerca: Comprar
                    buyFromNPC(npc)
                end
            end
        end
        wait(0.5)  -- Delay para no spamear (ajusta si quieres más rápido)
    end
end)

-- Re-conectar en respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
end)

print("🛒 AutoBuyNPC ACTIVADO! Cambia getgenv().AutoBuyNPC = false para parar.")
print("💡 Configura NPC_NAME o NPC_TAG arriba.")
