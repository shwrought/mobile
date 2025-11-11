-- ========================================
-- AUTO MULTI 150 RZ$ + LOOP HASTA COMPRAR (Toggle + Velocidad DEFAULT)
-- Compra TODOS los "150 RZ$" → Para CADA uno: LOOP INFINITO hasta COMPRA ÉXITO
-- ========================================

getgenv().AutoBuy150RZ = true  -- TOGGLE: true = COMPRA, false = PAUSA

local TARGET_PRICE = "150 RZ$"  -- ← NUEVO PRECIO
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

-- Encontrar TODOS los "150 RZ$"
local function findAll150RZ()
    local candidates = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local objText = obj.ObjectText
            -- Múltiples formatos: "150 RZ$", "150RZ$", etc.
            if objText:find("150") and objText:find("RZ$") then
                local model = obj:FindFirstAncestorOfClass("Model")
                if model and model:FindFirstChild("HumanoidRootPart") then
                    table.insert(candidates, {npc = model, prompt = obj})
                    print("🎯 Encontrado [" .. #candidates .. "]: " .. objText)
                end
            end
        end
    end
    print("📊 Total '150 RZ$' detectados: " .. #candidates)
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
                if not getgenv().AutoBuy150RZ then return end
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
    if not getgenv().AutoBuy150RZ then return end
    
    local attempt = 0
    pcall(function()
        prompt.HoldDuration = 0
        prompt.MaxActivationDistance = 50
    end)
    
    while getgenv().AutoBuy150RZ do
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
            print("✅ ¡COMPRADO ÉXITO 150 RZ$! Prompt eliminado.")
            return true  -- ÉXITO, pasa al siguiente NPC
        end
        
        wait(0.3)  -- Pausa entre loops
    end
    
    return false  -- Toggle off
end

-- LOOP PRINCIPAL INFINITO
spawn(function()
    while true do
        if getgenv().AutoBuy150RZ then
            local allTargets = findAll150RZ()
            if #allTargets > 0 then
                sortByDistance(allTargets)
                
                for i, target in ipairs(allTargets) do
                    if not getgenv().AutoBuy150RZ then break end
                    
                    local hrp = target.npc.HumanoidRootPart
                    local dist = (rootpart.Position - hrp.Position).Magnitude
                    
                    print("🎯 Target " .. i .. "/" .. #allTargets .. " 150 RZ$ (" .. math.floor(dist) .. " studs)")
                    
                    if dist > BUY_DISTANCE then
                        print("→ Caminando...")
                        walkToNPC(target.npc)
                    end
                    
                    print("🔥 INICIANDO LOOP HASTA COMPRAR 150 RZ$...")
                    loopUntilBuy(target.prompt)  -- ¡NO PASA AL SIGUIENTE HASTA COMPRAR!
                    
                    wait(1)  -- Cooldown entre NPCs
                end
                
                print("✅ Ciclo TODOS 150 RZ$ completado, re-busca...")
            else
                print("🔍 Sin '150 RZ$'... esperando.")
                wait(1)
            end
        else
            print("⏸️ PAUSADO. getgenv().AutoBuy150RZ = true para reanudar.")
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

print("🚀 AUTO 150 RZ$ + LOOP HASTA COMPRAR ACTIVADO!")
print("🔥 Para CADA NPC → LOOP INFINITO hasta que DESAPAREZCA el prompt")
print("📋 Toggle: getgenv().AutoBuy150RZ = true/false")
