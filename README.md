-- ========================================
-- AUTO MULTI $6,700,000 + TOGGLE DE COMPRA (Velocidad DEFAULT)
-- getgenv().AutoBuy6700k = true/false → Activa/Desactiva compra
-- ========================================

getgenv().AutoBuy6700k = true  -- TOGGLE: true = COMPRA, false = PAUSA

local TARGET_PRICE = "$250"
local BUY_DISTANCE = 15
local RETRY_ATTEMPTS = 2

-- Servicios
local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

-- Velocidad DEFAULT: NO TOCAR

-- Encontrar TODOS los $6.7M
local function findAll6700k()
    local candidates = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local objText = obj.ObjectText
            if objText:find(TARGET_PRICE) or 
               objText:find("6,700,000") or 
               objText:find("6700000") or 
               objText:find("6.7M") or
               objText:find("6,7M") then
                local model = obj:FindFirstAncestorOfClass("Model")
                if model and model:FindFirstChild("HumanoidRootPart") then
                    table.insert(candidates, {npc = model, prompt = obj})
                    print("Encontado [" .. #candidates .. "]: " .. objText)
                end
            end
        end
    end
    print("Total $6.7M detectados: " .. #candidates)
    return candidates
end

-- Ordenar por distancia
local function sortByDistance(candidates)
    table.sort(candidates, function(a, b)
        local distA = (rootpart.Position - a.npc.HumanoidRootPart.Position).Magnitude
        local distB = (rootpart.Position - b.npc.HumanoidRootPart.Position).Magnitude
        return distA < distB
    end)
end

-- Caminar NORMAL
local function walkToNPC(npc)
    local hrp = npc.HumanoidRootPart
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
                if not getgenv().AutoBuy6700k then return end  -- Respeta toggle
                humanoid:MoveTo(wp.Position)
                if wp.Action == Enum.PathWaypointAction.Jump then
                    humanoid.Jump = true
                end
                humanoid.MoveToFinished:Wait(4)
            end
        end
    end)
end

-- COMPRA CON RETRY
local function buyWithRetry(prompt)
    if not getgenv().AutoBuy6700k then return end  -- Respeta toggle
    
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 50
    end)
    
    for i = 1, RETRY_ATTEMPTS do
        if not getgenv().AutoBuy6700k then return end
        fireproximityprompt(prompt, 0)
        wait(0.3)
        print("Intento " .. i .. "/" .. RETRY_ATTEMPTS .. " → " .. prompt.ObjectText)
    end
    
    print("FINALIZADO compra para " .. prompt.ObjectText)
end

-- LOOP INFINITO CON TOGGLE
spawn(function()
    while true do
        if getgenv().AutoBuy6700k then
            local allTargets = findAll6700k()
            if #allTargets > 0 then
                sortByDistance(allTargets)
                
                for i, target in ipairs(allTargets) do
                    if not getgenv().AutoBuy6700k then break end  -- Pausa inmediata
                    
                    local hrp = target.npc.HumanoidRootPart
                    local dist = (rootpart.Position - hrp.Position).Magnitude
                    
                    print("Target " .. i .. "/" .. #allTargets .. " a " .. math.floor(dist) .. " studs")
                    
                    if dist > BUY_DISTANCE then
                        print("→ Caminando...")
                        walkToNPC(target.npc)
                    end
                    
                    print("→ Comprando...")
                    buyWithRetry(target.prompt)
                    
                    wait(0.8)
                end
                
                print("Ciclo completado, re-busca...")
            else
                print("Sin $6.7M... buscando.")
                wait(0.5)
            end
        else
            print("PAUSADO (toggle OFF). Cambia getgenv().AutoBuy6700k = true para reanudar.")
            wait(1)
        end
        
        wait(0.2)
    end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
end)

-- INICIO
print("TOGGLE AUTO $6,700,000 ACTIVADO!")
print("→ getgenv().AutoBuy6700k = true  → COMPRA")
print("→ getgenv().AutoBuy6700k = false → PAUSA")
print("→ Compra TODOS, velocidad default, retry 2x")
