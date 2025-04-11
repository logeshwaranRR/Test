@Override
public DashboardOrderInfoResponseDTO mapRequest(IOMInboundCallsBaseDTO baseDTO) throws IomMerchantException {
    DashboardOrderInfoRequestDTO requestDTO = (DashboardOrderInfoRequestDTO) baseDTO;
    String cacheKey = "dashboardOrderInfo:" + requestDTO.getOrderId();

    try {
        // 1. Check if list exists in cache
        Object cachedObject = redisCacheService.retrieveFromCache(cacheKey);
        if (cachedObject != null && cachedObject instanceof List) {
            List<DashboardOrderInfoByProductSoftware> cachedList =
                (List<DashboardOrderInfoByProductSoftware>) cachedObject;

            // 2. Get latest modified date from DB
            Date dbLatestModifiedDate = dashboardOrderInfoByProductSoftwareRepository
                .getLatestModifiedDate(requestDTO.getOrderId());

            // 3. Compare with cached modified date
            Date cacheModifiedDate = getLatestModifiedDateFromList(cachedList);

            if (dbLatestModifiedDate != null && cacheModifiedDate != null &&
                !dbLatestModifiedDate.after(cacheModifiedDate)) {

                // 4. Data is fresh — use cache
                DashboardOrderInfoResponseDTO responseDTO = new DashboardOrderInfoResponseDTO();
                dashboardOrderInfoMapper.mapQueryResultsToResponse(cachedList, responseDTO);
                return responseDTO;
            }

            // 5. Stale — continue to DB fetch
        }
    } catch (Exception e) {
        LOGGER.warn("Cache check failed, proceeding with DB. Key: {}", cacheKey, e);
    }

    // 6. Cache miss or stale, hit DB
    List<DashboardOrderInfoByProductSoftware> queryResults =
        dashboardOrderInfoByProductSoftwareRepository.getOrderDetailsFromRepo(
            requestDTO.getOrderId(), requestDTO.getBusinessPartyId(), requestDTO.getTaxId(),
            requestDTO.getBusinessName(), requestDTO.getMid(), requestDTO.getUnderwritingReferenceNumber(),
            requestDTO.getMcn(), requestDTO.getTransactionType(), requestDTO.getStatus(),
            requestDTO.getGuarantorPartyId(), requestDTO.getLineOfBusinesses(), requestDTO.getCommonAssociateIdentifier()
        );

    // 7. Add fresh list to cache
    try {
        redisCacheService.addToCache(cacheKey, queryResults);
    } catch (ServiceException e) {
        LOGGER.warn("Failed to update cache for key: {}", cacheKey, e);
    }

    DashboardOrderInfoResponseDTO responseDTO = new DashboardOrderInfoResponseDTO();
    dashboardOrderInfoMapper.mapQueryResultsToResponse(queryResults, responseDTO);
    return responseDTO;
}

// Helper method to extract latest modifiedDate from cached list
private Date getLatestModifiedDateFromList(List<DashboardOrderInfoByProductSoftware> list) {
    return list.stream()
        .map(DashboardOrderInfoByProductSoftware::getModifiedDate)
        .filter(Objects::nonNull)
        .max(Date::compareTo)
        .orElse(null);
}




@Query("SELECT MAX(e.modifiedDate) FROM DashboardOrderInfoByProductSoftware e WHERE e.orderId = :orderId")
Date getLatestModifiedDate(@Param("orderId") Long orderId);
