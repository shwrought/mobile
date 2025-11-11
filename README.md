-- ========================================
-- AUTO RECRUIT 150 Z$ (Luis R y cualquier 150 Z$) - NOVIEMBRE 2025
-- Toggle + Loop hasta comprar + Velocidad normal
-- ========================================

getgenv().AutoRecruit150Z = true  -- true = ON | false = OFF

local BUY_DISTANCE = 15
local RETRY_ATTEMPTS = 3

local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

-- ENCONTRAR TODOS LOS 150 Z$
local function findAll150Z()
    local targets = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local text = obj.ObjectText
            local action = obj.ActionText
            
            if (text:find("150 Z$") or text:find("150Z$") or text:find("150 Z")) 
            and (action == "Recruit" or action:find("Recruit") or action:find("Reclutar")) then
                
                local model = obj:FindFirstAncestorWhichIsA("Model")
                if model and model:FindFirstChild("HumanoidRootPart") then
                    table.insert(targets, {npc = model, prompt = obj})
                    print("Encontrado 150 Z$: " .. text .. " | " .. model.Name)
                end
            end
        end
    end
    print("Total 150 Z$ encontrados: " .. #targets)
    return targets
end

-- CAMINAR NORMAL
local function walkTo(npc)
    local hrp = npc.HumanoidRootPart
    local path = PathfindingService:CreatePath({AgentRadius = 3, AgentHeight = 6, AgentCanJump = true})
    
    pcall(function()
        path:ComputeAsync(rootpart.Position, hrp.Position + Vector3.new(0,0,-3))
        if path.Status == Enum.PathStatus.Success then
            for _, wp in ipairs(path:GetWaypoints()) do
                if not getgenv().AutoRecruit150Z then return end
                humanoid:MoveTo(wp.Position)
                if wp.Action == Enum.PathWaypointAction.Jump then humanoid.Jump = true end
                humanoid.MoveToFinished:Wait(5)
            end
        end
    end)
end

-- LOOP HASTA COMPRAR 100%
local function recruitUntilBought(prompt)
    pcall(function()
        prompt.HoldDuration = 0
        prompt.RequiresLineOfSight = false
        prompt.MaxActivationDistance = 50
    end)
    
    while getgenv().AutoRecruit150Z and prompt and prompt.Parent do
        for i = 1, RETRY_ATTEMPTS do
            fireproximityprompt(prompt, 0)
            wait(0.25)
        end
        print("Intentando reclutar: " .. prompt.ObjectText .. " (loop activo)")
        wait(0.8)
    end
    
    if not prompt.Parent then
        print("¡RECLUTADO CON ÉXITO! → " .. prompt.ObjectText)
    end
end

-- LOOP PRINCIPAL
spawn(function()
    while true do
        wait(0.5)
        if getgenv().AutoRecruit150Z then
            local targets = findAll150Z()
            if #targets > 0 then
                for i, target in ipairs(targets) do
                    if not getgenv().AutoRecruit150Z then break end
                    
                    local dist = (rootpart.Position - target.npc.HumanoidRootPart.Position).Magnitude
                    print("Reclutando " .. i .. "/" .. #targets .. " → " .. target.prompt.ObjectText .. " (" .. math.floor(dist) .. " studs)")
                    
                    if dist > BUY_DISTANCE then
                        walkTo(target.npc)
                    end
                    
                    recruitUntilBought(target.prompt)
                    wait(1.5)
                end
            else
                print("Esperando 150 Z$ en pasarela/dealer...")
            end
        else
            print("AUTO-RECLUTAMIENTO PAUSADO | Activa con: getgenv().AutoRecruit150Z = true")
            wait(2)
        end
    end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
end)

print("AUTO RECRUIT 150 Z$ ACTIVADO!")
print("Toggle → getgenv().AutoRecruit150Z = true/false")
