# ------------------------------------------------------------------
# physx
# ------------------------------------------------------------------
set(PHYSX_DIR
        ${3RD_PARTY_DIR}/physx
        PARENT_SCOPE)

# ------------------------------------------------------------------
# Includes
# ------------------------------------------------------------------
set(EVOENGINE_PHYSICS_3RD_PARTY_INCLUDES
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/physx/include
        PARENT_SCOPE
        )