-- ========================================
-- AUTO MULTI $6,700,000 + LOOP HASTA COMPRAR (Toggle + Velocidad DEFAULT)
-- Compra TODOS → Para CADA uno: LOOP INFINITO hasta que COMPRA ÉXITO
-- ========================================

getgenv().AutoBuy6700k = true  -- TOGGLE: true = COMPRA, false = PAUSA

local TARGET_PRICE = "$1,200,000"
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
                    print("🎯 Encontrado [" .. #candidates .. "]: " .. objText)
                end
            end
        end
    end
    print("📊 Total $6.7M detectados: " .. #candidates)
    return candidates
end

-- Ordenar por distancia (cerca → lejos)
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
                if not getgenv().AutoBuy6700k then return end
                humanoid:MoveTo(wp.Position)
                if wp.Action == Enum.PathWaypointAction.Jump then
                    humanoid.Jump = true
                end
                humanoid.MoveToFinished:Wait(4)
            end
        end
    end)
end

-- 🔥 LOOP INFINITO HASTA COMPRAR (para CADA NPC)
local function loopUntilBuy(prompt)
    if not getgenv().AutoBuy6700k then return end
    
    local attempt = 0
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 50
    end)
    
    while getgenv().AutoBuy6700k do
        attempt = attempt + 1
        
        -- Retry 2x por ciclo
        for i = 1, RETRY_ATTEMPTS do
            fireproximityprompt(prompt, 0)
            wait(0.2)
        end
        
        print("🔄 LOOP #" .. attempt .. " → Intentando comprar: " .. prompt.ObjectText)
        
        -- Chequea si DESAPARECIÓ el prompt (¡COMPRA ÉXITO!)
        wait(0.5)
        if not prompt or not prompt.Parent then
            print("✅ ¡COMPRADO ÉXITO! Prompt eliminado.")
            return true  -- ÉXITO, pasa al siguiente NPC
        end
        
        wait(0.3)  -- Pausa entre loops
    end
    
    return false  -- Toggle off
end

-- LOOP PRINCIPAL INFINITO
spawn(function()
    while true do
        if getgenv().AutoBuy6700k then
            local allTargets = findAll6700k()
            if #allTargets > 0 then
                sortByDistance(allTargets)
                
                for i, target in ipairs(allTargets) do
                    if not getgenv().AutoBuy6700k then break end
                    
                    local hrp = target.npc.HumanoidRootPart
                    local dist = (rootpart.Position - hrp.Position).Magnitude
                    
                    print("🎯 Target " .. i .. "/" .. #allTargets .. " (" .. math.floor(dist) .. " studs)")
                    
                    if dist > BUY_DISTANCE then
                        print("→ Caminando...")
                        walkToNPC(target.npc)
                    end
                    
                    print("🔥 INICIANDO LOOP HASTA COMPRAR...")
                    loopUntilBuy(target.prompt)  -- ¡NO PASA AL SIGUIENTE HASTA COMPRAR!
                    
                    wait(1)  -- Cooldown entre NPCs
                end
                
                print("✅ Ciclo TODOS completado, re-busca...")
            else
                print("🔍 Sin $6.7M... esperando.")
                wait(1)
            end
        else
            print("⏸️ PAUSADO. getgenv().AutoBuy6700k = true para reanudar.")
            wait(1)
        end
        
        wait(0.3)
    end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
end)

print("🚀 AUTO $6,700,000 + LOOP HASTA COMPRAR ACTIVADO!")
print("🔥 Para CADA NPC → LOOP INFINITO hasta que DESAPAREZCA el prompt")
print("📋 Toggle: getgenv().AutoBuy6700k = true/false")
