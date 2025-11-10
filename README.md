-- ========================================
-- AUTO COMPRAR $6,700,000 BRAINROT (Pasarela)
-- Detecta SOLO "$6,700,000" + "Comprar" → Va y COMPRA AUTO
-- ========================================

getgenv().AutoBuy6700k = true  -- Toggle: true=ON

local TARGET_PRICE = "$6,700,000"  -- ← CAMBIA AQUÍ SI QUIERES OTRO
local BUY_DISTANCE = 22
local SPEED = 120  -- Más rápido para brainrots caros
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

-- Encontrar MÁS CERCA $6,700,000
local function findClosest6700k()
    local candidates = {}
    
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local objText = obj.ObjectText
            local actText = obj.ActionText
            
            -- Detecta EXACTO: "$6,700,000" + "Comprar"
            if objText == TARGET_PRICE and (actText == "Comprar" or actText == "Buy") then
                local model = obj:FindFirstAncestorOfClass("Model")
                if model and model:FindFirstChild("HumanoidRootPart") then
                    table.insert(candidates, {npc = model, prompt = obj})
                    print("RARE Encontrado " .. TARGET_PRICE .. "!")
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

-- Ir al NPC (rápido y preciso)
local function goToNPC(npc)
    local hrp = npc:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local targetPos = hrp.Position + Vector3.new(0, 4, 0)
    
    local path = PathfindingService:CreatePath({
        AgentRadius = 3,
        AgentHeight = 6,
        AgentCanJump = true,
        WaypointSpacing = 3
    })
    
    local success = pcall(function() path:ComputeAsync(rootpart.Position, targetPos) end)
    
    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for _, wp in ipairs(waypoints) do
            if not getgenv().AutoBuy6700k then return end
            humanoid:MoveTo(wp.Position)
            if wp.Action == Enum.PathWaypointAction.Jump then humanoid.Jump = true end
            humanoid.MoveToFinished:Wait(1.8)
        end
    else
        -- Tween directo
        local dist = (rootpart.Position - targetPos).Magnitude
        local tweenInfo = TweenInfo.new(math.max(dist / 100, 0.4), "Linear")
        local tween = TweenService:Create(rootpart, tweenInfo, {CFrame = CFrame.new(targetPos)})
        tween:Play()
        tween.Completed:Wait()
    end
end

-- COMPRAR
local function buyFromPrompt(prompt)
    fireproximityprompt(prompt)
    print("JACKPOT COMPRADO " .. TARGET_PRICE .. " BRAINROT!")
    wait(1.2)  -- Cooldown para no spamear
end

-- LOOP INFINITO
spawn(function()
    while true do
        if getgenv().AutoBuy6700k then
            local target = findClosest6700k()
            if target then
                local hrp = target.npc:FindFirstChild("HumanoidRootPart")
                local dist = (rootpart.Position - hrp.Position).Magnitude
                
                print("TARGET " .. TARGET_PRICE .. " a " .. math.floor(dist) .. " studs")
                
                if dist > BUY_DISTANCE then
                    goToNPC(target.npc)
                else
                    buyFromPrompt(target.prompt)
                end
            else
                print("WAITING Esperando " .. TARGET_PRICE .. " en pasarela...")
                wait(1.5)
            end
        end
        wait(0.15)  -- Ultra rápido
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

print("JACKPOT AUTO " .. TARGET_PRICE .. " ACTIVADO!")
print("Para parar: getgenv().AutoBuy6700k = false")
