-- ========================================
-- AUTO COMPRAR [PRECIO] BRAINROTS - Configurable
-- Detecta CUALQUIER precio → Va y COMPRA AUTO 24/7
-- ========================================

getgenv().AutoBuyPrice = true  -- Toggle: true=ON

local TARGET_PRICE = "$250"  -- ← CAMBIA AQUÍ: "$500", "$100", "$1000", etc.
local BUY_DISTANCE = 20
local SPEED = 100
local JUMP_POWER = 100

-- Servicios
local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

humanoid.WalkSpeed = SPEED
humanoid.JumpPower = JUMP_POWER

-- Encontrar MÁS CERCA [TU PRECIO]
local function findClosestPrice()
    local candidates = {}
    
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local objText = obj.ObjectText:lower()
            local actText = obj.ActionText:lower()
            
            -- Detecta TU PRECIO + "comprar"
            if objText:find(TARGET_PRICE:lower()) and (actText:find("comprar") or actText:find("buy")) then
                local model = obj:FindFirstAncestorOfClass("Model")
                if model and model:FindFirstChild("HumanoidRootPart") then
                    table.insert(candidates, {npc = model, prompt = obj})
                    print("🔍 Encontrado " .. TARGET_PRICE .. ": " .. obj.ObjectText)
                end
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

-- Ir al NPC
local function goToNPC(npc)
    local hrp = npc:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local targetPos = hrp.Position + Vector3.new(0, 3, 0)
    
    local path = PathfindingService:CreatePath({
        AgentRadius = 3,
        AgentHeight = 6,
        AgentCanJump = true,
        WaypointSpacing = 4
    })
    
    local success = pcall(function() path:ComputeAsync(rootpart.Position, targetPos) end)
    
    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for _, wp in ipairs(waypoints) do
            if not getgenv().AutoBuyPrice then return end
            humanoid:MoveTo(wp.Position)
            if wp.Action == Enum.PathWaypointAction.Jump then humanoid.Jump = true end
            humanoid.MoveToFinished:Wait(2)
        end
    else
        local dist = (rootpart.Position - targetPos).Magnitude
        local tweenInfo = TweenInfo.new(math.max(dist / 80, 0.5), "Linear")
        local tween = TweenService:Create(rootpart, tweenInfo, {CFrame = CFrame.new(targetPos)})
        tween:Play()
        tween.Completed:Wait()
    end
end

-- COMPRAR
local function buyFromPrompt(prompt)
    fireproximityprompt(prompt)
    print("✅ ¡COMPRADO " .. TARGET_PRICE .. " BRAINROT!")
    return true
end

-- LOOP INFINITO
spawn(function()
    while true do
        if getgenv().AutoBuyPrice then
            local target = findClosestPrice()
            if target then
                local hrp = target.npc:FindFirstChild("HumanoidRootPart")
                local dist = (rootpart.Position - hrp.Position).Magnitude
                
                print("🛒 " .. TARGET_PRICE .. " a " .. math.floor(dist) .. " studs")
                
                if dist > BUY_DISTANCE then
                    goToNPC(target.npc)
                else
                    buyFromPrompt(target.prompt)
                    wait(0.8)
                end
            else
                print("🔍 Buscando " .. TARGET_PRICE .. " en pasarela...")
                wait(1)
            end
        end
        wait(0.2)
    end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
    humanoid.WalkSpeed = SPEED
    humanoid.JumpPower = JUMP_POWER
end)

print("🚀 AUTO " .. TARGET_PRICE .. " ACTIVADO!")
print("Para parar: getgenv().AutoBuyPrice = false")
