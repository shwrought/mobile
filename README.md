-- Client: AutoGotoAndBuy (StarterPlayerScripts, LocalScript)
local CollectionService = game:GetService("CollectionService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local BuyEvent = ReplicatedStorage:WaitForChild("BuyNPC")

-- CONFIG
local TARGET_NAME = "Noobini Lusinini" -- si quieres buscar un nombre concreto; si no, usa nil
local SEARCH_TAG = "Comun"
local REACH_DISTANCE = 3 -- distancia para considerar "llegado" (studs)

local function isTarget(npc)
    if not npc or not npc:IsA("Model") then return false end
    if not CollectionService:HasTag(npc, SEARCH_TAG) then return false end
    if TARGET_NAME and string.upper(npc.Name) ~= string.upper(TARGET_NAME) then
        return false
    end
    -- opcional: más validaciones (humanoid, price, etc)
    return true
end

local function findNearestTarget()
    local best, bestDist = nil, math.huge
    for _, npc in ipairs(CollectionService:GetTagged(SEARCH_TAG)) do
        if npc and npc:IsDescendantOf(workspace) and npc:IsA("Model") and isTarget(npc) then
            local npcRoot = npc:FindFirstChild("HumanoidRootPart") or npc.PrimaryPart
            if npcRoot then
                local d = (hrp.Position - npcRoot.Position).Magnitude
                if d < bestDist then
                    bestDist = d
                    best = npc
                end
            end
        end
    end
    return best, bestDist
end

local function gotoAndBuy(npc)
    if not npc then return end
    local npcRoot = npc:FindFirstChild("HumanoidRootPart") or npc.PrimaryPart
    if not npcRoot then return end

    -- Pedir al humano que se mueva hacia la posición
    humanoid:MoveTo(npcRoot.Position)

    -- esperar hasta que llegue (o timeout)
    local reached = humanoid.MoveToFinished:Wait() -- true si llegó, false si falló
    -- alternativa: esperar a que la distancia sea menor
    local arrived = (hrp.Position - npcRoot.Position).Magnitude <= REACH_DISTANCE

    if arrived or reached then
        -- solicitud segura al servidor
        BuyEvent:FireServer(npc)
        print("[Client] Solicitud de compra enviada para:", npc.Name)
    else
        print("[Client] No se llegó al NPC:", npc.Name)
    end
end

-- Ejemplo: al presionar una tecla o comando ejecuta la búsqueda y compra
-- Aquí lo hacemos con la tecla "B" (puedes adaptarlo)
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.B then
        local target, dist = findNearestTarget()
        if target then
            print("[Client] Objetivo encontrado:", target.Name, "dist:", dist)
            gotoAndBuy(target)
        else
            print("[Client] No se encontró ningún NPC con etiqueta '"..SEARCH_TAG.."' y nombre objetivo.")
        end
    end
end)

-- Alternativa: Autostart (buscar y proceder inmediatamente)
// --[[ Si prefieres que busque y vaya sin presionar tecla, descomenta:
-- task.spawn(function()
--     while true do
--         local t = findNearestTarget()
--         if t then
--             gotoAndBuy(t)
--             break -- o sleep para repetir
--         end
--         task.wait(1)
--     end
-- end)
--]]
